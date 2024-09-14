from web3 import Web3
import json
import random

# Connect to Ganache
w3 = Web3(Web3.HTTPProvider("http://127.0.0.1:8545"))
w3.eth.default_account = w3.eth.accounts[0]

# Load compiled contract
with open('TradeLifecycle_sol_TradeLifecycle.bin', 'r') as bin_file:
    bytecode = bin_file.read()

with open('TradeLifecycle_sol_TradeLifecycle.abi', 'r') as abi_file:
    abi = json.load(abi_file)

# Deploy contract
#TradeLifecycle = w3.eth.contract(abi=abi, bytecode=bytecode)
#tx_hash = TradeLifecycle.constructor().transact()
#tx_receipt = w3.eth.wait_for_transaction_receipt(tx_hash)
#print("contract created")
#print(tx_receipt)

#contract = w3.eth.contract(address=tx_receipt.contractAddress, abi=abi)
contract_address = "0x9Fe4D782D5Eb2050023eFF407A1fAdb5425D6031"
contract = w3.eth.contract(address=contract_address, abi=abi)

# Function to get trade audit trail
def get_trade_audit(trade_id):
    events = contract.events.TradeCreated.create_filter(from_block=0, argument_filters={'id': trade_id}).get_all_entries()
    events += contract.events.TradeConfirmed.create_filter(from_block=0, argument_filters={'id': trade_id}).get_all_entries()
    events += contract.events.TradeSettled.create_filter(from_block=0, argument_filters={'id': trade_id}).get_all_entries()
    events += contract.events.TradeCanceled.create_filter(from_block=0, argument_filters={'id': trade_id}).get_all_entries()

    audit_trail = []
    for event in events:
        audit_trail.append({
            'event': event['event'],
            'tradeId': event['args']['id'],
            'details': event['args'].get('details', None),
            'initiator': event['args'].get('initiator', None),
            'blockNumber': event['blockNumber'],
            'transactionHash': event['transactionHash'].hex(),
        })
    return audit_trail


trade_id = random.randint(1, 100)

# Create trade
tx_hash = contract.functions.createTrade(trade_id, "Trade details").transact()
tx_receipt = w3.eth.wait_for_transaction_receipt(tx_hash)
print("trade created")
print(tx_receipt)

# Confirm trade
tx_hash = contract.functions.confirmTrade(trade_id).transact()
tx_receipt = w3.eth.wait_for_transaction_receipt(tx_hash)
print("trade confirm")
print(tx_receipt)

# Settle trade
tx_hash = contract.functions.settleTrade(trade_id).transact()
tx_receipt = w3.eth.wait_for_transaction_receipt(tx_hash)
print("settle trade")
print(tx_receipt)

# Get audit trail for trade ID 1
audit_trail = get_trade_audit(trade_id)
for event in audit_trail:
    print(f"Event: {event['event']}, Trade ID: {event['tradeId']}, Block: {event['blockNumber']}, TxHash: {event['transactionHash']}")

