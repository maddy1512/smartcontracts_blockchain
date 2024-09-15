// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract TradeLifecycle {
    enum TradeStatus { Created, Confirmed, Settled, Canceled }

    struct Trade {
        uint256 id;
        string details;
        TradeStatus status;
    }

    mapping(uint256 => Trade) public trades;
    uint256 public tradeCount;

    event TradeCreated(uint256 id);
    event TradeConfirmed(uint256 id);
    event TradeSettled(uint256 id);
    event TradeCanceled(uint256 id);

    function createTrade(uint256 _tradeid, string memory _details) public {
        trades[_tradeid] = Trade(_tradeid, _details, TradeStatus.Created);
        emit TradeCreated(_tradeid);
    }

    function confirmTrade(uint256 _tradeId) public {
        require(trades[_tradeId].status == TradeStatus.Created, "Trade must be in Created state.");
        trades[_tradeId].status = TradeStatus.Confirmed;
        emit TradeConfirmed(_tradeId);
    }

    function settleTrade(uint256 _tradeId) public {
        require(trades[_tradeId].status == TradeStatus.Confirmed, "Trade must be Confirmed.");
        trades[_tradeId].status = TradeStatus.Settled;
        emit TradeSettled(_tradeId);
    }

    function cancelTrade(uint256 _tradeId) public {
        require(trades[_tradeId].status != TradeStatus.Settled, "Cannot cancel Settled trade.");
        trades[_tradeId].status = TradeStatus.Canceled;
        emit TradeCanceled(_tradeId);
    }

    function getTrade(uint256 _tradeId) public view returns (uint256, string memory, TradeStatus) {
        Trade memory t = trades[_tradeId];
        return (t.id, t.details, t.status);
    }
}

