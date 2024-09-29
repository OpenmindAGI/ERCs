---
title: Identity and Governance Interface for Human/Robot Societies 
description: This ERC defines standardized interfaces for managing the identities of humans and robots, establishing and maintaining rule sets (“laws”) that apply to those societies, and standardizing the immigration (“joining”) and emigration (“departure”) of humans and robots from the rights and responsibilities of those rule sets.
author: OpenMind, Jan Liphardt <jan@openmind.org>, Shaohong Zhong (ShaohongZ) <shaohong@openmind.org>, Boyuan Chen (bchen-dev) <boyuan@openmind.org>, Paige Xu <paige@openmind.org>
discussions-to: <URL>
status: Draft
type: Standards Track
category: ERC
created: 2024-09-29
---

## Abstract

This proposal defines two core interfaces: IUniversalIdentity and IUniversalCharter, providing mechanisms for humans, and robots to establish their identities and to create decentralized communities governed by specific rule sets. The IUniversalIdentity interface establishes the fair and equitable treatment of sentient computer architectures other than the human brain, enabling robots to acquire on-chain identities, and thereby interact and transact with humans. The IUniversalCharter enables humans and robots to create, join (“immigrate”), maintain, leave (“emigrate”), and end self-regulated societies based on predefined rule sets, providing a framework for collaboration and prosperity for mixed societies of humans and robots. These interfaces aim to provide a flexible yet enforceable structure for human-robot interactions in decentralized systems, ensuring efficiency, transparency, and security for all participants.

## Motivation

The human brain is a wet, massively parallel electrochemical computer. Recent hardware and software advances make it likely that soon, human societies will need tools for interacting with sentient, non-human computers, such as robots. Our current forms of government, where citizens are auto-enrolled into specific rule sets depending on where they were born, do not gracefully map onto robots without a traditional birthplace or birthtime. Among many difficulties being experienced by robots, they are (currently) unable to obtain standard forms of ID (such as passports), it is not clear which rule sets apply to them (since in general they are not born in specific places), and they cannot currently use the standard human-centered banking system. Likewise, in the event in which robots are harmed by humans or non-biological computers, it is not clear which human court has jurisdiction. 

Traditional geographically-defined and human-centered systems can be inefficient, slow to change, opaque, and can struggle to accommodate global, virtualized societies.  Decentralized, immutable, and public computers offer an ideal solution to these limitations, since they do not inherently discriminate against non-human computers and therefore offer an equitable and more just framework for governance.  In particular, smart contracts can provide a powerful framework for regulating the rights and responsibilities or interacting parties regardless of implementation details of their compute architecture.

The general motivation of this ERC is to provide a standard interface for smart contracts focusing on identity/governance for heterogeneous global societies. While there are an unlimited number of such rule sets, there are obvious benefits to providing a standard interface to those rule sets, greatly reducing the friction and complexity of creating, joining, maintaining, and ending such societies. The specific motivation of this ERC is twofold:

1. Robot Identity Creation and Management: To participate meaningfully and comply with on-chain laws, non-humans such as robots must be able to acquire meaningful on-chain identities. Importantly, these identities should enable robots to enjoy the benefits of, but also bear the responsibility of, being part of a specific society. Thus, we propose to enable smart contract-based identity for robots. Specifically, each robot is represented by a smart contract and needs to follow the rules defined in the contract to interact with other agents on the chain. This interface also ensures flexibility by all participants to propose, adopt, or revoke rules, enabling self-managed compliance and transparent interaction with other participants.

2. Rule Creation and Enforcement: For humans and robots to effectively collaborate, they must agree upon a rule set. This Ethereum-based system provides a basic decentralized framework for governing human-robot interactions through smart contracts. We propose to enforce the rule-sets by requiring humans and robots to join regulated access smart contracts that check their compliance with the given rules. We also ensure scalability, whereby multiple regulated access contracts can be created to tailor to different purposes, and humans and robots can choose to join the relevant system as needed. 

Together, these interfaces form the foundation for managing complex human-robot interactions, enabling a decentralized, verifiable, and rule-based ecosystem where robots and humans can interact securely, transparently, and responsibly, for maximum benefit of all.


## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

```solidity
interface IUniversalIdentity {

    /// @notice Adds a rule to the robot's identity, showing that the robot agrees to follow the rule.
    /// @param rule The dynamic byte array representing the rule that the robot agrees to follow.
    /// @dev The rule SHOULD come from the rule sets defined in the IUniversalCharter contract that the robot intends to join. 
    /// @dev This function SHOULD be implemented by contracts to add the rules that the robot intends to follow.
    function addRule(bytes memory rule) external;

    /// @notice Removes a rule from the robot's identity.
    /// @param rule The dynamic byte array representing the rule that the robot no longer agrees to follow.
    /// @dev This function SHOULD be implemented by contracts to remove the rules that the robot does not intend to follow.
    function removeRule(bytes memory rule) external;

    /// @notice Checks if the robot complies with a specific rule.
    /// @param rule The rule to check.
    /// @return bool Returns true if the robot complies with the rule.
    /// @dev This function MUST be implemented by contracts for compliance verification.
    function checkCompliance(bytes memory rule) external view returns (bool);

    /// @dev Emitted when a rule is added to the robot's identity.
    event RuleAdded(bytes rule);

    /// @dev Emitted when a rule is removed from the robot's identity.
    event RuleRemoved(bytes rule);
    
    /// @dev Emitted when a charter is subscribed to.
    event SubscribedToCharter(address indexed charter);

    /// @dev Emitted when it is unsubscribed from a charter.
    event UnsubscribedFromCharter(address indexed charter);
}

interface IUniversalCharter {

    // Define the user types as an enum.
    enum UserType { Human, Robot }

    /// @notice Registers a user (human or robot) to join the system by agreeing to a rule set.
    /// @param userType The type of user.
    /// @param ruleSet The array of individual rules the user agrees to follow. 
    /// @dev This function MUST be implemented by contracts using this interface.
    /// @dev The implementing contract MUST ensure that the user complies with the specified rule set before registering them in the system by invoking `checkCompliance`.
    function registerUser(UserType userType, bytes[] memory ruleSet) external;

    /// @notice Allows a user (human or robot) to leave the system 
    /// @dev This function MUST be callable only by the user themselves (via `msg.sender`). 
    /// @dev The implementing contract MUST ensure that the user has complied with all necessary rules before they can successfully leave the system by invoking `checkCompliance`.
    function leaveSystem() external;

    /// @notice Checks if the user (human or robot) complies with the system’s rules.
    /// @param user Address of the user (human or robot).
    /// @param ruleSet The array of individual rules to verify. 
    /// @return bool Returns true if the user complies with the rule set.
    /// @dev This function SHOULD invoke the `checkCompliance` function of the user’s IUniversalIdentity contract to check for rules individually. 
    /// @dev This function MUST be implemented by contracts for compliance verification.
    function checkCompliance(address user, bytes[] memory ruleSet) external view returns (bool);

    /// @notice Updates the rule set.
    /// @param newRuleSet The array of new individual rules replacing the existing ones. 
    /// @dev This function SHOULD be restricted to authorized users (e.g., contract owner).
    function updateRuleSet(bytes[] memory newRuleSet) external;

    /// @notice Terminates the contract, preventing any further registrations or interactions.
    /// @dev This function SHOULD be restricted to authorized users (e.g., contract owner).
    /// @dev This function SHOULD be implemented by contracts.
    function terminateContract() external;

    /// @dev Emitted when a user joins the system by agreeing to a set of rules.
    event UserRegistered(address indexed user, UserType userType, bytes[] ruleSet);

    /// @dev Emitted when a user successfully leaves the system after fulfilling obligations.
    event UserLeft(address indexed user);

    /// @dev Emitted when a user’s compliance with a rule set is verified.
    event ComplianceChecked(address indexed user, bytes[] ruleSet);

    /// @dev Emitted when a rule set is updated.
    event RuleSetUpdated(bytes[] newRuleSet, address updatedBy);
}

```

## Rationale

**IUniversalIdentity**

`addRule(bytes memory rule)`
This function allows a robot to add a specific rule to its identity, signaling its intention to comply with that rule. It supports dynamic rule management, enabling robots to flexibly adopt new compliance requirements as they join different systems.

`removeRule(bytes memory rule)`
This function removes a rule from the robot’s identity, typically when the robot no longer needs to comply with that rule (e.g., leaving a system). It allows contracts to dynamically manage the robot’s obligations, ensuring its rule set remains accurate and up-to-date.

`checkCompliance(bytes memory rule)`
This function checks whether the robot complies with a specific rule. It ensures that robots are adhering to rules before being granted permission to perform any actions

`Events (RuleAdded, RuleRemoved)`
These events provide transparency and traceability, making it easier to track compliance status.

**IUniversalCharter**

`enum UserType { Human, Robot }`
The UserType enum distinguishes between humans and robots in a gas-efficient manner, making it easier for contracts to handle different user types without the cost and errors associated with strings. This provides the basis for differentiated handling in future implementations, allowing the system to potentially apply different rules or logic based on whether the user is a human or a robot.

`registerUser(UserType userType, bytes[] memory ruleSet)`
This function is critical for registering users, linking them to a specific rule set defined. It ensures that a user—whether human or robot—is bound to a particular set of rules upon joining the system.

`leaveSystem()`
This function allows users to leave the system by de-registering themselves. The use of msg.sender ensures that only the user themselves can invoke this function, enhancing security. A compliance check is required (via checkCompliance) before the user is allowed to leave, ensuring that users fulfill all obligations before exiting the system.

`checkCompliance(address user, bytes[] memory ruleSet)`
This function checks if a user is in compliance with a specific rule set they agreed to follow. It ensures that the system can efficiently manage and verify compliance against predefined rule sets, helping maintain the overall integrity of the system.

`updateRuleSet(bytes[] memory newRuleSet)`
This function enables the modification of rule sets in the contract, ensuring that the system can adapt over time. 

`terminateContract()`
This function allows for the permanent shutdown of the contract, preventing any further interactions or registrations within the system. 

`Events (UserRegistered, UserLeft, ComplianceChecked, RuleSetUpdated, ContractTerminated)`
These events collectively ensure that key activities are visible to off-chain systems and participants, making the system auditable and transparent.

## Backwards Compatibility

No backward compatibility issues found.

## Test Cases

## Reference Implementation

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import { OwnableUpgradeable } from "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
import { UniversalCharter } from "./UniversalCharter.sol";

// Interfaces
import { IUniversalIdentity } from "./interface/IUniversalIdentity.sol";
import { IUniversalCharter } from "./interface/IUniversalCharter.sol";

/// @title UniversalIdentity
/// @notice The UniversalIdentity contract is used to manage the identity of robots.

contract UniversalIdentity is IUniversalIdentity, OwnableUpgradeable {
    /// @notice Version identifier for the current implementation of the contract.
    string public constant VERSION = "v0.0.1";

    /// @notice Mapping to store rules that the robot has agreed to follow
    mapping(bytes => bool) private robotRules;

    /// @notice Mapping to store the off-chain compliance status for each rule
    mapping(bytes => bool) private complianceStatus;

    /// @notice Track the charters the robot is subscribed to
    mapping(address => bool) private subscribedCharters;

    /// @notice Custom errors to save gas on reverts
    error RuleNotAgreed(bytes rule);
    error RuleNotCompliant(bytes rule);
    error RuleAlreadyAdded(bytes rule);

    /// @dev Event to emit when compliance is checked
    /// @param updater The address of the compliance updater (owner of the contract)
    /// @param rule The rule that was checked
    event ComplianceChecked(address indexed updater, bytes rule);

    /// @notice Modifier to check if a rule exists
    modifier ruleExists(bytes memory rule) {
        require(robotRules[rule], "Rule does not exist");
        _;
    }

    /// @notice Constructor to set the owner
    constructor() {
        initialize({ _owner: address(0xdEaD) });
    }

    /// @dev Initializer function
    function initialize(address _owner) public initializer {
        __Ownable_init();
        transferOwnership(_owner);
    }

    /// @notice Adds a rule to the robot's identity
    /// @param rule The dynamic byte array representing the rule that the robot agrees to follow.
    function addRule(bytes memory rule) external override onlyOwner {
        if (robotRules[rule]) {
            revert RuleAlreadyAdded(rule);
        }

        // Add rule to the mapping
        robotRules[rule] = true;

        emit RuleAdded(rule);
    }

    /// @notice Removes a rule from the robot's identity
    /// @param rule The dynamic byte array representing the rule that the robot no longer agrees to follow.
    function removeRule(bytes memory rule) external override onlyOwner ruleExists(rule) {
        robotRules[rule] = false;
        complianceStatus[rule] = false;

        emit RuleRemoved(rule);
    }

    /// @notice Subscribe and register to a specific UniversalCharter contract using its stored rule set
    /// @param charter The address of the UniversalCharter contract
    /// @param version The version of the rule set to fetch and register for
    function subscribeAndRegisterToCharter(address charter, uint256 version) external {
        require(!subscribedCharters[charter], "Already subscribed to this charter");
        subscribedCharters[charter] = true;

        // Fetch the rule set directly from the UniversalCharter contract using the public getter
        bytes[] memory ruleSet = UniversalCharter(charter).getRuleSet(version);

        // Register as a robot in the charter using the fetched rule set
        UniversalCharter(charter).registerUser(IUniversalCharter.UserType.Robot, ruleSet);

        emit SubscribedToCharter(charter);
    }

    /// @notice Leave the system for a specific UniversalCharter contract
    /// @param charter The address of the UniversalCharter contract to leave
    function leaveCharter(address charter) external {
        require(subscribedCharters[charter], "Not subscribed to this charter");

        // Call the leaveSystem function of the UniversalCharter contract
        UniversalCharter(charter).leaveSystem();

        // Unsubscribe from the charter after leaving
        subscribedCharters[charter] = false;
        emit UnsubscribedFromCharter(charter);
    }

    /// @notice Updates compliance status for a rule (called by the owner)
    /// @param rule The dynamic byte array representing the rule
    /// @param status The compliance status (true if compliant, false if not)
    function updateCompliance(bytes memory rule, bool status) external onlyOwner ruleExists(rule) {
        complianceStatus[rule] = status;

        emit ComplianceChecked(msg.sender, rule);
    }

    /// @notice Checks if the robot has agreed to follow a specific rule and if it is compliant
    /// @param rule The rule to check.
    /// @return bool Returns true if the robot has agreed to the rule and is compliant
    function checkCompliance(bytes memory rule) external view override returns (bool) {
        if (!robotRules[rule]) {
            revert RuleNotAgreed(rule);
        }

        return true;
    }

    /// @notice Gets the compliance status of a rule
    /// @param rule The rule to check.
    function getRule(bytes memory rule) external view returns (bool) {
        return robotRules[rule];
    }

    /// @notice Gets the subscription status of a charter
    /// @param charter The address of the charter to check.
    function getSubscribedCharters(address charter) external view returns (bool) {
        return subscribedCharters[charter];
    }

    /// @notice Gets the compliance status of a rule
    /// @param rule The rule to check.
    function getComplianceStatus(bytes memory rule) external view returns (bool) {
        return complianceStatus[rule];
    }
}
```

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.20;

import { OwnableUpgradeable } from "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
import { SystemConfig } from "./SystemConfig.sol";

// Interfaces
import { IUniversalCharter } from "./interface/IUniversalCharter.sol";
import { IUniversalIdentity } from "./interface/IUniversalIdentity.sol";

/// @title UniversalCharter
/// @notice The UniversalCharter contract is used to manage the registration and compliance of users.

contract UniversalCharter is IUniversalCharter, OwnableUpgradeable {
    /// @notice Struct to store information about a registered user
    struct UserInfo {
        bool isRegistered;
        UserType userType;
        uint256 ruleSetVersion; // Rule set version the user is following
    }

    /// @notice Mapping to store registered users
    mapping(address => UserInfo) private users;

    /// @notice Mapping to store rule sets by version number
    mapping(uint256 => bytes[]) private ruleSets;

    /// @notice Mapping to track the rule set hash and its corresponding version
    mapping(bytes32 => uint256) private ruleSetVersions;

    /// @notice Version identifier for the current implementation of the contract.
    string public constant VERSION = "v0.0.1";

    /// @notice Variable to track the current version of the rule set
    uint256 private currentVersion;

    /// @notice Variable to store the address of the SystemConfig contract
    SystemConfig public systemConfig;

    /// @notice Error for when a method cannot be called when paused. This could be renamed
    /// to `Paused` in the future, but it collides with the `Paused` event.
    error CallPaused();

    /// @notice Reverts when paused.
    modifier whenNotPaused() {
        if (paused()) revert CallPaused();
        _;
    }

    /// @notice Constucts the UniversalCharter contract.
    constructor() {
        initialize({ _owner: address(0xdEaD), _systemConfig: address(0xdEaD) });
    }

    /// @dev Initializer function
    function initialize(address _owner, address _systemConfig) public initializer {
        __Ownable_init();
        transferOwnership(_owner);
        systemConfig = SystemConfig(_systemConfig);
    }

    /// @notice Registers a user (either human or robot) by agreeing to a rule set
    /// @param userType The type of user: Human or Robot
    /// @param ruleSet The array of individual rules the user agrees to follow
    function registerUser(UserType userType, bytes[] memory ruleSet) external override whenNotPaused {
        require(!users[msg.sender].isRegistered, "User already registered");

        // Hash the rule set to find the corresponding version
        bytes32 ruleSetHash = keccak256(abi.encode(ruleSet));
        uint256 version = ruleSetVersions[ruleSetHash];
        require(version > 0, "Invalid or unregistered rule set");

        // For robots, ensure compliance with each rule via the UniversalIdentity contract
        if (userType == UserType.Robot) {
            require(_checkRobotCompliance(msg.sender, version), "Robot not compliant with rule set");
        }

        // Register the user with the versioned rule set
        users[msg.sender] = UserInfo({ isRegistered: true, userType: userType, ruleSetVersion: version });

        emit UserRegistered(msg.sender, userType, ruleSet);
    }

    /// @notice Allows a user (human or robot) to leave the system after passing compliance checks
    function leaveSystem() external override whenNotPaused {
        require(users[msg.sender].isRegistered, "User not registered");

        UserInfo memory userInfo = users[msg.sender];

        // For robots, verify compliance with all rules in the rule set
        uint256 version = userInfo.ruleSetVersion;
        if (userInfo.userType == UserType.Robot) {
            require(_checkRobotCompliance(msg.sender, version), "Robot not compliant with rule set");
        }

        users[msg.sender] = UserInfo({ isRegistered: false, userType: UserType.Human, ruleSetVersion: 0 });

        emit UserLeft(msg.sender);
    }

    /// @notice Checks if a user complies with their registered rule set
    /// @param user The address of the user (human or robot)
    /// @param ruleSet The array of individual rules to verify
    /// @return bool Returns true if the user complies with the given rule set
    function checkCompliance(address user, bytes[] memory ruleSet) external view override returns (bool) {
        require(users[user].isRegistered, "User not registered");

        // Hash the provided rule set to find the corresponding version
        bytes32 ruleSetHash = keccak256(abi.encode(ruleSet));
        uint256 version = ruleSetVersions[ruleSetHash];
        require(version > 0, "Invalid or unregistered rule set");
        require(users[user].ruleSetVersion == version, "Rule set version mismatch");

        // For robots, check compliance with each rule in the UniversalIdentity contract
        if (users[user].userType == UserType.Robot) {
            return _checkRobotCompliance(user, version);
        }

        // If the user is human, compliance is assumed for now (can be extended)
        return true;
    }

    /// @notice Internal function to check compliance for robots with their rule set version
    /// @dev This function will revert if the robot is not compliant with any rule. Returns true for view purposes.
    /// @param robotAddress The address of the robot
    /// @param version The version of the rule set to verify compliance with
    /// @return bool Returns true if the robot is compliant with all the rules in the rule set
    function _checkRobotCompliance(address robotAddress, uint256 version) internal view returns (bool) {
        IUniversalIdentity robot = IUniversalIdentity(robotAddress);
        bytes[] memory rules = ruleSets[version];

        for (uint256 i = 0; i < rules.length; i++) {
            if (!robot.checkCompliance(rules[i])) {
                return false;
            }
        }

        return true;
    }

    /// @notice Updates or defines a new rule set version.
    /// @param newRuleSet The array of new individual rules.
    /// @dev This function SHOULD be restricted to authorized users (e.g., contract owner).
    function updateRuleSet(bytes[] memory newRuleSet) external whenNotPaused onlyOwner {
        require(newRuleSet.length > 0, "Cannot update to an empty rule set");

        // Hash the new rule set and ensure it's not already registered
        bytes32 ruleSetHash = keccak256(abi.encode(newRuleSet));
        require(ruleSetVersions[ruleSetHash] == 0, "Rule set already registered");

        // Increment the version and store the new rule set
        currentVersion += 1;
        ruleSets[currentVersion] = newRuleSet;
        ruleSetVersions[ruleSetHash] = currentVersion;

        emit RuleSetUpdated(newRuleSet, msg.sender);
    }

    /// @notice Getter for the latest version of the rule set.
    function getLatestRuleSetVersion() external view returns (uint256) {
        return currentVersion;
    }

    /// @notice Get the rule set for a specific version.
    /// @param version The version of the rule set to retrieve.
    function getRuleSet(uint256 version) external view returns (bytes[] memory) {
        return ruleSets[version];
    }

    /// @notice Get the version number for a specific rule set.
    /// @param ruleSet The hash of the rule set to retrieve the version for.
    function getRuleSetVersion(bytes32 ruleSet) external view returns (uint256) {
        return ruleSetVersions[ruleSet];
    }

    function getUserInfo(address user) external view returns (UserInfo memory) {
        return users[user];
    }

    /// @notice Getter for the current paused status.
    /// @return paused_ Whether or not the contract is paused.
    function paused() public view returns (bool paused_) {
        paused_ = systemConfig.paused();
    }
}
```

## Security Considerations

Compliance Updater: The compliance updater role in the UniversalIdentity contract is critical for updating compliance statuses (currently limited to the owner). It is essential to ensure secure ownership to minimize the risks of unauthorized or malicious updates. 

Rule Management: Functions such as addRule, removeRule, and updateCompliance in the UniversalIdentity contract and updateRuleSet in the UniversalCharter contract directly affect rule enforcement. It’s essential to ensure these functions are only callable by authorized users.

Upgradeable Contracts: The use of OwnableUpgradeable introduces risks during the initialization and upgrade process. Ensuring that the initialize function is protected against re-execution is critical to avoid reinitialization attacks.

Gas Consumption: Excessively large rule sets could lead to high gas costs or DoS risks. Consider setting limits on the number of rules allowed per rule set to maintain gas efficiency and avoid performance issues.

## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).