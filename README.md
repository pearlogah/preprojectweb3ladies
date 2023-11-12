# preprojectweb3ladies

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract TimelockVault is ReentrancyGuard, Ownable {
    using SafeERC20 for IERC20;

    IERC20 public token;
    uint256 public releaseBlock;

    event Deposit(address indexed account, uint256 amount);
    event Withdrawal(address indexed account, uint256 amount);

    constructor(IERC20 _token, uint256 _releaseBlock) Ownable(msg.sender) {
        token = _token;
        releaseBlock = block.number + _releaseBlock;
    }

    function deposit(uint256 _amount) external nonReentrant {
        require(_amount > 0, "Amount must be greater than 0");
        require(token.balanceOf(msg.sender) >= _amount, "Insufficient token balance");
        token.safeTransferFrom(msg.sender, address(this), _amount);
        emit Deposit(msg.sender, _amount);
    }

    function withdraw() external nonReentrant onlyOwner {
        require(block.number >= releaseBlock, "Release block not reached");
        uint256 balance = token.balanceOf(address(this));
        token.safeTransfer(owner(), balance);
        emit Withdrawal(owner(), balance);
    }

    function getReleaseBlock() external view returns (uint256) {
        return releaseBlock;
    }
}
