// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract TradeLifecycle {
    struct Trade {
        uint id;
        string details;
        address initiator;
        bool isConfirmed;
        bool isSettled;
        bool isCanceled;
        bool frontOfficeApproved;
        bool middleOfficeApproved;
        bool backOfficeApproved;
    }

    uint public tradeCounter = 0;
    mapping(uint => Trade) public trades;

    address public frontOffice;
    address public middleOffice;
    address public backOffice;

    event TradeCreated(uint id, address initiator);
    event TradeCancellationRequested(uint id, address requestedBy);
    event TradeCancelled(uint id);
    event TradeCancelledByOffice(uint id, string office);

    modifier onlyFrontOffice() {
        require(msg.sender == frontOffice, "Only the front office can call this function");
        _;
    }

    modifier onlyMiddleOffice() {
        require(msg.sender == middleOffice, "Only the middle office can call this function");
        _;
    }

    modifier onlyBackOffice() {
        require(msg.sender == backOffice, "Only the back office can call this function");
        _;
    }

    constructor(address _frontOffice, address _middleOffice, address _backOffice) {
        frontOffice = _frontOffice;
        middleOffice = _middleOffice;
        backOffice = _backOffice;
    }

    function createTrade(string memory _details) public {
        tradeCounter++;
        trades[tradeCounter] = Trade({
            id: tradeCounter,
            details: _details,
            initiator: msg.sender,
            isConfirmed: false,
            isSettled: false,
            isCanceled: false,
            frontOfficeApproved: false,
            middleOfficeApproved: false,
            backOfficeApproved: false
        });

        emit TradeCreated(tradeCounter,msg.sender);
    }

    // Front Office can initiate cancellation
    function requestTradeCancellation(uint _tradeId) public onlyFrontOffice {
        Trade storage trade = trades[_tradeId];
        require(!trade.isCanceled, "Trade already canceled");
        trade.frontOfficeApproved = true;
        emit TradeCancellationRequested(_tradeId, msg.sender);
    }

    // Middle Office validates trade cancellation
    function middleOfficeApproval(uint _tradeId) public onlyMiddleOffice {
        Trade storage trade = trades[_tradeId];
        require(trade.frontOfficeApproved, "Front office has not approved cancellation");
        trade.middleOfficeApproved = true;
        emit TradeCancelledByOffice(_tradeId, "Middle Office");
    }

    // Back Office gives final approval
    function backOfficeApproval(uint _tradeId) public onlyBackOffice {
        Trade storage trade = trades[_tradeId];
        require(trade.middleOfficeApproved, "Middle office has not approved cancellation");

        // All approvals are done, cancel the trade
        trade.isCanceled = true;
        emit TradeCancelledByOffice(_tradeId, "Back Office");
        emit TradeCancelled(_tradeId);
    }

    // Check if trade is canceled
    function isTradeCanceled(uint _tradeId) public view returns (bool) {
        return trades[_tradeId].isCanceled;
    }
}

