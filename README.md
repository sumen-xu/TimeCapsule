# TimeCapsule
BaseTimeCapsule.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.34;

contract BaseTimeCapsule {
    address public immutable owner;
    uint256 public capsuleFee = 0.0005 ether;

    struct Capsule {
        address creator;
        string message;
        uint256 unlockTime;
        bool isOpened;
    }

    Capsule[] public capsules;

    event CapsuleCreated(uint256 indexed id, address creator, uint256 unlockTime);
    event CapsuleOpened(uint256 indexed id, address opener);

    constructor() {
        owner = msg.sender;
    }

    // 创建时间胶囊
    function createCapsule(string memory _message, uint256 _unlockTime) external payable {
        require(msg.value == capsuleFee, "Must pay exactly 0.0005 ETH");
        require(bytes(_message).length > 0 && bytes(_message).length <= 500, "Message 1-500 chars");
        require(_unlockTime > block.timestamp + 1 days, "Unlock time must be at least 1 day later");

        capsules.push(Capsule({
            creator: msg.sender,
            message: _message,
            unlockTime: _unlockTime,
            isOpened: false
        }));

        emit CapsuleCreated(capsules.length - 1, msg.sender, _unlockTime);
    }

    // 打开胶囊（时间到了才能打开）
    function openCapsule(uint256 _capsuleId) external {
        require(_capsuleId < capsules.length, "Capsule not exist");
        Capsule storage c = capsules[_capsuleId];
        require(!c.isOpened, "Already opened");
        require(block.timestamp >= c.unlockTime
