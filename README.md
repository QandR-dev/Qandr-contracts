# Quiz Platform Smart Contract

This smart contract implements a quiz platform where users can create quizzes with USDT prize pools, claim rewards, and participate in daily questions.

## Features

- Create quizzes with USDT prize pools
- ETH fee for quiz creation
- Prize distribution for top 10 winners
- Daily question attempt tracking
- ETH fee for daily question attempts
- Admin controls for fee management

## Prize Distribution

The prize pool is distributed among the top 10 winners as follows:
- 1st Place: 25%
- 2nd Place: 20%
- 3rd Place: 15%
- 4th Place: 10%
- 5th-6th Place: 7.5% each
- 7th-8th Place: 5% each
- 9th-10th Place: 2.5% each

## Smart Contract Functions

### Quiz Management
- `createQuiz(uint256 prizeAmount)`: Create a new quiz with USDT prize pool
- `announceWinners(uint256 quizId, address[] winnerAddresses)`: Set winners for a quiz
- `claimPrize(uint256 quizId, uint256 rank)`: Claim prize for winners

### Daily Question
- `attemptDailyQuestion()`: Attempt daily question (requires ETH fee)
- `canAttemptDaily(address user)`: Check if user can attempt daily question

### Admin Functions
- `setCreationFee(uint256 _newFee)`: Update quiz creation fee
- `setDailyQuestionFee(uint256 _newFee)`: Update daily question fee
- `withdrawFees()`: Withdraw collected ETH fees
- `recoverTokens()`: Emergency function to recover USDT
- `updateUSDTAddress(address _newUSDT)`: Update USDT token address


## Requirements

- Solidity ^0.8.0
- OpenZeppelin Contracts
- USDT Token Contract

## License

MIT License
