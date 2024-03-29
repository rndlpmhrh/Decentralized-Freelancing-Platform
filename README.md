# Decentralized-Freelancing-Platform
打造一个基于WEB3的自由职业者平台，利用智能合约来简化付款和争议解决。
from web3 import Web3
from solcx import compile_source

# Connect to Ethereum node
w3 = Web3(Web3.HTTPProvider('YOUR_INFURA_OR_ALCHEMY_HTTPS_URL'))

# Check connection
if not w3.isConnected():
    print("Failed to connect to Ethereum node.")
    exit()

# Compile Smart Contract (Simplified Example)
contract_source_code = '''
pragma solidity ^0.8.0;

contract FreelanceContract {
    address public employer;
    address public freelancer;
    uint public contractAmount;
    bool public jobCompleted = false;

    constructor(address _freelancer, uint _contractAmount) payable {
        employer = msg.sender;
        freelancer = _freelancer;
        contractAmount = _contractAmount;
    }

    function markJobCompleted() public {
        require(msg.sender == employer, "Only the employer can mark the job as completed.");
        jobCompleted = true;
    }

    function withdrawFunds() public {
        require(jobCompleted, "Job is not yet completed.");
        require(msg.sender == freelancer, "Only the freelancer can withdraw funds.");
        payable(freelancer).transfer(contractAmount);
    }
}
'''

# Compile Contract
compiled_sol = compile_source(contract_source_code, output_values=['abi', 'bin'])
contract_id, contract_interface = compiled_sol.popitem()

# Get bytecode / abi
bytecode = contract_interface['bin']
abi = contract_interface['abi']

# Deploy Contract
FreelanceContract = w3.eth.contract(abi=abi, bytecode=bytecode)
tx_hash = FreelanceContract.constructor(w3.toChecksumAddress('FREELANCER_ADDRESS'), w3.toWei(1, 'ether')).transact({'from': w3.eth.accounts[0], 'value': w3.toWei(1, 'ether')})

# Wait for the transaction to be mined
tx_receipt = w3.eth.wait_for_transaction_receipt(tx_hash)

# Contract Instance
contract_instance = w3.eth.contract(address=tx_receipt.contractAddress, abi=abi)

# Example Usage
# Employer marks job as completed
tx_hash = contract_instance.functions.markJobCompleted().transact({'from': w3.eth.accounts[0]})
w3.eth.wait_for_transaction_receipt(tx_hash)

# Freelancer withdraws funds
tx_hash = contract_instance.functions.withdrawFunds().transact({'from': 'FREELANCER_ADDRESS'})
w3.eth.wait_for_transaction_receipt(tx_hash)

print("Demo completed successfully.")
