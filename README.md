README:

##InsurancePolicyManagement Smart Contract
Overview
The InsurancePolicyManagement smart contract facilitates the management of insurance policies on the Ethereum blockchain. It allows users to purchase policies, file claims, cancel policies, and manages funds securely. This contract is written in Solidity, the programming language for Ethereum smart contracts.
Description

##Features
Contract Structure

Insurer: Address of the insurer who manages the contract.
Policy Struct: Defines the structure of an insurance policy including ID, insured address, premium, coverage amount, and status.
Mappings: Tracks policies by ID and by policy holder.
Events: Logs important contract events such as policy purchase, claim, cancellation, and insurer change.
Functions

purchasePolicy: Allows users to purchase insurance policies by sending the required premium.
addPolicy: Enables the insurer to add a policy for a specific insured party.
fileClaim: Allows policyholders to file claims to receive coverage.
cancelPolicy: Permits policyholders to cancel their policies and receive premium refunds.
changeInsurer: Allows the current insurer to transfer control to a new address.
withdrawFunds: Enables the insurer to withdraw accumulated funds from the contract.
Error Handling

Modifiers: Includes onlyInsurer modifier to restrict certain functions to the insurer.
Require Statements: Ensures conditions are met before executing critical operations, preventing unauthorized actions.
Example Function

exampleErrorHandling: Demonstrates usage of revert for error handling within a transaction.
Payment Handling

receive() function: Allows the contract to receive Ether payments from policy premiums and claims.
Usage

##Deployment:

Deploy the contract on an Ethereum network compatible with Solidity version ^0.8.0.
Set the initial insurer address during deployment to ensure proper contract ownership.
Interacting with the Contract:

Policy Purchase: Call purchasePolicy function with the required premium and coverage amount.
Policy Management: Use addPolicy to add policies, changeInsurer to change the insurer address, and withdrawFunds to manage funds.
Policyholder Actions: Utilize fileClaim to claim coverage and cancelPolicy to cancel policies.
Security Considerations
Ownership: Ensure the initial insurer address is secure and not easily compromised.
Function Permissions: Utilize modifiers (onlyInsurer) and require statements to enforce access control.
Testing: Thoroughly test all contract functionalities, including edge cases and error conditions, before deploying on mainnet.
Development Environment
Solidity: Version ^0.8.0 recommended.
Development Tools: Use Remix IDE, Truffle, or Hardhat for local development and testing.
Testing: Deploy on test networks (e.g., Rinkeby, Ropsten) using tools like Ganache for comprehensive testing.
Getting Started / Execution
Setting Up Development Environment:

Install Remix IDE, Truffle, or Hardhat depending on your preference for local development.
Ensure Solidity compiler version ^0.8.0 is installed.
Deploying the Contract:

Use your preferred development tool to compile and deploy the contract to an Ethereum test network (e.g., Ganache, Rinkeby, Ropsten).
Specify the initial insurer address during deployment.
Interacting with the Contract:

Use the deployed contract address to interact via the functions described.
Test each function thoroughly in various scenarios to ensure correctness and security.
Testing:
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract InsurancePolicyManagement {
    address public insurer;
    uint256 public nextPolicyId;

    struct Policy {
        uint256 policyId;
        address insured;
        uint256 premium;
        uint256 coverageAmount;
        bool isActive;
        bool hasClaimed;
    }

    mapping(uint256 => Policy) public policies;
    mapping(address => uint256[]) public policyHolderToPolicies;

    event PolicyPurchased(uint256 indexed policyId, address indexed insured, uint256 premium, uint256 coverageAmount);
    event PolicyClaimed(uint256 indexed policyId, address indexed insured, uint256 amount);
    event PolicyCancelled(uint256 indexed policyId, address indexed insured);
    event InsurerChanged(address indexed oldInsurer, address indexed newInsurer);
    event PolicyAdded(uint256 indexed policyId, address indexed insured, uint256 premium, uint256 coverageAmount);

    modifier onlyInsurer() {
        require(msg.sender == insurer, "Only the insurer can perform this action");
        _;
    }

    constructor() {
        insurer = msg.sender;
    }

    /**
     * @dev Allows a user to purchase an insurance policy by sending the required premium.
     * @param _premium The premium amount for the insurance policy.
     * @param _coverageAmount The coverage amount for the insurance policy.
     */
    function purchasePolicy(uint256 _premium, uint256 _coverageAmount) external payable {
        require(msg.value == _premium, "Incorrect premium amount sent");

        policies[nextPolicyId] = Policy({
            policyId: nextPolicyId,
            insured: msg.sender,
            premium: _premium,
            coverageAmount: _coverageAmount,
            isActive: true,
            hasClaimed: false
        });

        policyHolderToPolicies[msg.sender].push(nextPolicyId);

        emit PolicyPurchased(nextPolicyId, msg.sender, _premium, _coverageAmount);

        nextPolicyId++;
    }

    /**
     * @dev Allows the insurer to add a policy directly.
     * @param _insured The address of the insured.
     * @param _premium The premium amount for the insurance policy.
     * @param _coverageAmount The coverage amount for the insurance policy.
     */
    function addPolicy(address _insured, uint256 _premium, uint256 _coverageAmount) external onlyInsurer {
        require(_insured != address(0), "Invalid insured address");
        require(_premium > 0, "Premium amount must be greater than 0");

        policies[nextPolicyId] = Policy({
            policyId: nextPolicyId,
            insured: _insured,
            premium: _premium,
            coverageAmount: _coverageAmount,
            isActive: true,
            hasClaimed: false
        });

        policyHolderToPolicies[_insured].push(nextPolicyId);

        emit PolicyAdded(nextPolicyId, _insured, _premium, _coverageAmount);

        nextPolicyId++;
    }

    /**
     * @dev Allows a policyholder to file a claim and receive the coverage amount.
     * @param _policyId The ID of the policy to claim.
     */
    function fileClaim(uint256 _policyId) external {
        Policy storage policy = policies[_policyId];
        require(policy.insured == msg.sender, "You are not the insured party");
        require(policy.isActive, "Policy is not active");
        require(!policy.hasClaimed, "Policy already claimed");

        uint256 claimAmount = policy.coverageAmount;
        policy.hasClaimed = true;
        policy.isActive = false;

        // Ensure the policy is updated correctly
        assert(!policy.isActive && policy.hasClaimed);

        payable(msg.sender).transfer(claimAmount);

        emit PolicyClaimed(_policyId, msg.sender, claimAmount);
    }

    /**
     * @dev Allows a policyholder to cancel their policy and receive a refund of the premium.
     * @param _policyId The ID of the policy to cancel.
     */
    function cancelPolicy(uint256 _policyId) external {
        Policy storage policy = policies[_policyId];
        require(policy.insured == msg.sender, "You are not the insured party");
        require(policy.isActive, "Policy is already inactive");

        uint256 refundAmount = policy.premium;
        policy.isActive = false;

        // Ensure the policy is updated correctly
        assert(!policy.isActive);

        payable(msg.sender).transfer(refundAmount);

        emit PolicyCancelled(_policyId, msg.sender);
    }

    /**
     * @dev Allows the insurer to change the insurer address.
     * @param _newInsurer The new insurer address.
     */
    function changeInsurer(address _newInsurer) external onlyInsurer {
        require(_newInsurer != address(0), "Invalid address");
        require(_newInsurer != insurer, "New insurer must be different from current one");
        
        address oldInsurer = insurer;
        insurer = _newInsurer;

        emit InsurerChanged(oldInsurer, _newInsurer);
    }

    /**
     * @dev Example function to demonstrate error handling using revert.
     */
    function exampleErrorHandling() external pure {
        revert("Operation failed, reverting transaction");
    }

    /**
     * @dev Allows the contract to receive Ether.
     */
    receive() external payable {}

    /**
     * @dev Allows the insurer to withdraw funds from the contract.
     * @param _amount The amount to withdraw.
     */
    function withdrawFunds(uint256 _amount) external onlyInsurer {
        require(address(this).balance >= _amount, "Insufficient contract balance");
        payable(insurer).transfer(_amount);
    }
} 



Authors
Anushka kumari
anushka.ak29@gmail.com
8168563516
