# rent-payment
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract RentPayment {
    address public landlord;
    address public tenant;
    uint256 public rentAmount;
    uint256 public rentDueDate;
    uint256 public rentInterval;
    bool public isRentPaid;

    event RentPaid(address indexed tenant, uint256 amount, uint256 date);
    event RentWithdrawn(address indexed landlord, uint256 amount, uint256 date);
    event RentDueDateUpdated(uint256 newDueDate);

    modifier onlyTenant() {
        require(msg.sender == tenant, "Only tenant can call this function.");
        _;
    }

    modifier onlyLandlord() {
        require(msg.sender == landlord, "Only landlord can call this function.");
        _;
    }

    modifier rentDue() {
        require(block.timestamp >= rentDueDate, "Rent is not due yet.");
        _;
    }

    constructor(address _tenant, uint256 _rentAmount, uint256 _rentInterval) {
        landlord = msg.sender;
        tenant = _tenant;
        rentAmount = _rentAmount;
        rentInterval = _rentInterval;
        rentDueDate = block.timestamp + rentInterval;
        isRentPaid = false;
    }

    function payRent() public payable onlyTenant rentDue {
        require(msg.value == rentAmount, "Incorrect rent amount.");
        isRentPaid = true;
        emit RentPaid(msg.sender, msg.value, block.timestamp);
    }

    function withdrawRent() public onlyLandlord {
        require(isRentPaid, "Rent has not been paid.");
        isRentPaid = false;
        rentDueDate += rentInterval;
        uint256 amount = address(this).balance;
        payable(landlord).transfer(amount);
        emit RentWithdrawn(msg.sender, amount, block.timestamp);
        emit RentDueDateUpdated(rentDueDate);
    }

    function getContractBalance() public view returns (uint256) {
        return address(this).balance;
    }
}
