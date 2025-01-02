// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract QuizRewards is Ownable, ReentrancyGuard {
    IERC20 public usdtToken;
    uint256 public creationFee;
    uint256 public dailyQuestionFee;
    uint256 public quizCount;

    struct Quiz {
        uint256 quizId;
        address creator;
        uint256 prizePool;
        bool isActive;
        bool isPrizeClaimed;
        mapping(uint256 => address) winners;
        mapping(address => bool) hasClaimed;
    }

    mapping(uint256 => Quiz) public quizzes;
    mapping(address => uint256) public lastQuestionSolveTime;

    uint256[10] public prizeDistribution = [
        2500, 2000, 1500, 1000, 750,
        750, 500, 500, 250, 250
    ];

    event QuizCreated(uint256 indexed quizId, address creator, uint256 prizePool);
    event WinnersAnnounced(uint256 indexed quizId, address[] winners);
    event PrizeClaimed(uint256 indexed quizId, address winner, uint256 amount);
    event CreationFeeUpdated(uint256 newFee);
    event DailyQuestionAttempted(address indexed user, uint256 timestamp);

    constructor(
        address initialOwner,
        address _usdtToken, 
        uint256 _creationFee, 
        uint256 _dailyQuestionFee
    ) Ownable(initialOwner) {
        usdtToken = IERC20(_usdtToken);
        creationFee = _creationFee;
        dailyQuestionFee = _dailyQuestionFee;
    }

    function setCreationFee(uint256 _newFee) external onlyOwner {
        creationFee = _newFee;
        emit CreationFeeUpdated(_newFee);
    }

    function setDailyQuestionFee(uint256 _newFee) external onlyOwner {
        dailyQuestionFee = _newFee;
    }

    function createQuiz(uint256 prizeAmount) external payable {
        require(prizeAmount > 0, "Prize must be greater than 0");
        require(msg.value >= creationFee, "Insufficient fee");
        
        require(
            usdtToken.transferFrom(msg.sender, address(this), prizeAmount),
            "Prize pool transfer failed"
        );
        
        uint256 quizId = quizCount++;
        Quiz storage newQuiz = quizzes[quizId];
        newQuiz.quizId = quizId;
        newQuiz.creator = msg.sender;
        newQuiz.prizePool = prizeAmount;
        newQuiz.isActive = true;
        
        emit QuizCreated(quizId, msg.sender, prizeAmount);
    }

    function announceWinners(uint256 quizId, address[] calldata winnerAddresses) external onlyOwner {
        require(winnerAddresses.length == 10, "Must provide exactly 10 winners");
        Quiz storage quiz = quizzes[quizId];
        require(quiz.isActive, "Quiz not active");
        
        for(uint256 i = 0; i < 10; i++) {
            quiz.winners[i] = winnerAddresses[i];
        }
        
        quiz.isActive = false;
        emit WinnersAnnounced(quizId, winnerAddresses);
    }

    function claimPrize(uint256 quizId, uint256 rank) external nonReentrant {
        Quiz storage quiz = quizzes[quizId];
        require(!quiz.isActive, "Quiz still active");
        require(!quiz.hasClaimed[msg.sender], "Already claimed");
        require(quiz.winners[rank] == msg.sender, "Not a winner at this rank");
        require(rank < 10, "Invalid rank");
        
        uint256 prizeAmount = (quiz.prizePool * prizeDistribution[rank]) / 10000;
        quiz.hasClaimed[msg.sender] = true;
        
        require(
            usdtToken.transfer(msg.sender, prizeAmount),
            "Prize transfer failed"
        );
        
        emit PrizeClaimed(quizId, msg.sender, prizeAmount);
    }

    function attemptDailyQuestion() external payable {
        require(msg.value >= dailyQuestionFee, "Insufficient fee");
        
        uint256 today = block.timestamp - (block.timestamp % 86400);
        require(lastQuestionSolveTime[msg.sender] < today, "Already attempted today");
        
        lastQuestionSolveTime[msg.sender] = block.timestamp;
        emit DailyQuestionAttempted(msg.sender, block.timestamp);
    }

    function canAttemptDaily(address user) external view returns (bool) {
        uint256 today = block.timestamp - (block.timestamp % 86400);
        return lastQuestionSolveTime[user] < today;
    }

    function getCreationFee() external view returns (uint256) {
        return creationFee;
    }

    function withdrawFees() external onlyOwner {
        (bool success, ) = payable(owner()).call{value: address(this).balance}("");
        require(success, "Withdraw failed");
    }

    function recoverTokens() external onlyOwner {
        uint256 balance = usdtToken.balanceOf(address(this));
        require(usdtToken.transfer(owner(), balance), "Transfer failed");
    }

    function updateUSDTAddress(address _newUSDT) external onlyOwner {
        usdtToken = IERC20(_newUSDT);
    }

    receive() external payable {}
}
