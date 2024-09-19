from web3 import Web3
import json

# Connect to Ethereum node
w3 = Web3(Web3.HTTPProvider("http://127.0.0.1:8545"))

# Set default account (front office for now)
w3.eth.default_account = w3.eth.accounts[0]  # Front office account
middle_office = w3.eth.accounts[1]  # Front office account
back_office = w3.eth.accounts[2]  # Front office account


# Load smart contract ABI and address
with open('Trade_sol_TradeLifecycle.abi', 'r') as abi_file:
    abi = json.load(abi_file)

with open('Trade_sol_TradeLifecycle.bin', 'r') as bin_file:
    bytecode = bin_file.read()



# TradeLifecycle = w3.eth.contract(abi=abi, bytecode=bytecode)
# tx_hash = TradeLifecycle.constructor(w3.eth.accounts[0], middle_office, back_office).transact()
# tx_receipt = w3.eth.wait_for_transaction_receipt(tx_hash)
# print("contract created")
# print(tx_receipt)
# contract_address = tx_receipt.contractAddress
# print(f"{contract_address}")

contract_address = '0x5fd7aa663d377E664F626E60c8e0F85c36049495'

contract = w3.eth.contract(address=contract_address, abi=abi)

# Function to create a trade
def create_trade(details):
    tx_hash = contract.functions.createTrade(details).transact()
    receipt = w3.eth.wait_for_transaction_receipt(tx_hash)
    print(f"Trade created with details: {details}")

# Function to request trade cancellation (front office)
def request_cancellation(trade_id):
    tx_hash = contract.functions.requestTradeCancellation(trade_id).transact()
    receipt = w3.eth.wait_for_transaction_receipt(tx_hash)
    print(f"Front office requested cancellation for trade: {trade_id}")

# Function for middle office to approve cancellation
def middle_office_approval(trade_id):
    w3.eth.default_account = w3.eth.accounts[1]  # Middle office account
    tx_hash = contract.functions.middleOfficeApproval(trade_id).transact()
    receipt = w3.eth.wait_for_transaction_receipt(tx_hash)
    print(f"Middle office approved cancellation for trade: {trade_id}")

# Function for back office to approve cancellation
def back_office_approval(trade_id):
    w3.eth.default_account = w3.eth.accounts[2]  # Back office account
    tx_hash = contract.functions.backOfficeApproval(trade_id).transact()
    receipt = w3.eth.wait_for_transaction_receipt(tx_hash)
    print(f"Back office approved cancellation for trade: {trade_id}")

# Example usage:
create_trade("Trade between A and B")

# Front office requests cancellation
request_cancellation(1)

# Middle office approves
middle_office_approval(1)

# Back office approves
back_office_approval(1)
