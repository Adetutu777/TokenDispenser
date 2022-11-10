<template>
<div class="body-col">
    <div class="container element-container">
        <!-- creating tabs, gotten from bootstrapVue -->
        <b-card no-body>
            <b-tabs pills card>
                <b-tab title="View Functions" active>
                    <b-card-text>
                        <div>
                            <b-card no-body>
                                <b-tabs pills card vertical nav-wrapper-class="w-50">
                                    <!-- total supply -->
                                    <b-tab title="Total Supply" active @click="getTotalSupply">
                                        <b-card-text>{{totalSupply}}</b-card-text>
                                    </b-tab>
                                    <!-- token name -->
                                    <b-tab title="Token Name" @click="getTokenName">
                                        <b-card-text>{{tokenName}}</b-card-text>
                                    </b-tab>
                                    <!-- token symbol -->
                                    <b-tab title="Token Symbol" @click="getTokenSymbol">
                                        <b-card-text>{{tokenSymbol}}</b-card-text>
                                    </b-tab>
                                    <!-- balance -->
                                    <b-tab title="Balance">
                                        <b-card-text>
                                            <span v-if="balance">{{balance}}</span>
                                            <b-form-input class="mb-4" v-model="myAddress" placeholder="Enter your address"></b-form-input>
                                            <b-button variant="primary" @click="getBalance">Get Balance</b-button>
                                        </b-card-text>
                                    </b-tab>
                                    <!-- allowance -->
                                    <b-tab title="Allowance">
                                        <b-card-text>
                                            <span>{{tokenAllowance}}</span>
                                            <b-form-input class="mb-4" v-model="spenderAddress" placeholder="Enter spender address"></b-form-input>
                                            <b-button @click="getAllowance">Get Allowance</b-button>
                                        </b-card-text>
                                    </b-tab>
                                </b-tabs>
                            </b-card>
                        </div>
                    </b-card-text>
                </b-tab>

                <!-- write functions -->
                <b-tab title="Write Functions">
                    <b-card-text>
                        <div>
                            <b-card no-body>
                                <b-tabs pills card vertical nav-wrapper-class="w-50">
                                    <!-- transfer -->
                                    <span v-if="checkMetaMaskConnected">

                                        <b-tab title="Transfer">
                                            <b-card-text>
                                           
                                           <div class="transfer bg-success text-light mb-3">{{transferMsg}}</div> 
                                                <span v-if="transfer">{{transferMsg}}</span>
                                                {{walletAddress}}
                                                <b-form-input class="mb-4" v-model="receiverAddress" placeholder="Enter receiver address"></b-form-input>
                                                <b-form-input class="mb-4" v-model="amount" placeholder="Amount to transfer"></b-form-input>
                                                <b-button variant="primary" @click="transferToken">Transfer</b-button>
                                            </b-card-text>
                                        </b-tab>

                                        <!-- approval -->
                                        <b-tab title="Approve">
                                            <b-card-text>
                                           
                                           <div class="transfer bg-success text-light mb-3">{{approvalMsg}}</div>                
                                                <b-button variant="primary" @click="approveToken">Approve</b-button>
                                            </b-card-text>
                                        </b-tab>
                                    </span>

                                    <span v-else>
                                        <b-button variant="primary" class="connect" @click="onClickConnect">Connect Wallet</b-button>
                                    </span>

                                </b-tabs>
                            </b-card>
                        </div>
                    </b-card-text>
                </b-tab>

            </b-tabs>
        </b-card>
    </div>
</div>
</template>

<script>
import {ref,onMounted } from '@nuxtjs/composition-api';
import {ethers} from "ethers";
import { ContractAddress, rpcUrl,spender } from "@/constant"
import ABI from "@/abi.json"
export default {
    setup() {
        const signer = ref('')
        const totalSupply = ref('')
        const tokenName = ref('')
        const tokenSymbol = ref('')
        const balance = ref('')
        const myAddress = ref('0x3dF043F83F81a273812C17859433CC15E55e0EDc')
        const tokenAllowance = ref('')
        const spenderAddress = ref(spender)
        const receiverAddress = ref('0xB5f3815f527285B061AC1d0897aB928701Bb9a71')
        const amount = ref('0.00001')
        const transferMsg = ref('')
        const walletAddress = ref('')
        const checkMetaMaskConnected = ref(false)
        const signerProvider = ref('')
        const approvalMsg = ref('')
       

        const provider = new ethers.providers.JsonRpcProvider(rpcUrl);

        onMounted(async () => {
            const signerOrProvider = new ethers.providers.Web3Provider(window.ethereum);
            signer.value = signerOrProvider?.getSigner()
            const account = await signerOrProvider?.listAccounts()
            walletAddress.value = account[0];
            signerProvider.value = signerOrProvider
        })

        const getContract = (isSigner = false, contractAddress = ContractAddress, abi = ABI, ) => {
            const newProvider = isSigner ? signer.value : provider;
            return new ethers.Contract(contractAddress, abi, newProvider);
        }
// function to get Total supply
        const getTotalSupply = async () => {
            const contract = getContract()
            totalSupply.value = 'loading'
            const totalTokenSupply = await contract.totalSupply()
            totalSupply.value = totalTokenSupply.toString() / 10 ** 18
        }
        // function to get Token Name
        const getTokenName = async () => {
            const contract = getContract()
            tokenName.value = 'loading'
            const nameToken = await contract.name()
            tokenName.value = nameToken
        }
// function to get symbol
        const getTokenSymbol = async () => {
            const contract = getContract()
            tokenSymbol.value = 'loading'
            const symbolToken = await contract.symbol()
            tokenSymbol.value = symbolToken
        }
// function to get balance
        const getBalance = async () => {
            const contract = getContract()
            balance.value = 'loading'
            const checkBalance = await contract.balanceOf(myAddress.value)
            balance.value = checkBalance.toString() / 10 ** 18
            myAddress.value = ''
        }

        // function to get allowance
        const getAllowance = async () => {
            const contract = getContract()
            tokenAllowance.value = 'loading'
            const checkAllowance = await contract.allowance(walletAddress.value, spender)
            tokenAllowance.value = checkAllowance.toString()
            // tokenAllowance.value = checkAllowance.toString() / 10 ** 18

        }
// function to connect to MetaMask
        const onClickConnect = async () => {
            try {
                // Will open the MetaMask UI
                const account = await signerProvider.value.send("eth_requestAccounts", []);
                checkMetaMaskConnected.value = true;
                walletAddress.value = account[0];
            } catch (error) {
                console.error(error);
                console.log(error.message)
            }
        };

        // transfer Token
        const transferToken = async () => {
            const contract = getContract(true)
            const amountToSend = Number(amount.value) * 10 ** 18
            const txn = await contract.transfer(receiverAddress.value, amountToSend.toString())
            transferMsg.value = 'sending'
            await txn.wait()
            transferMsg.value = `You have successfully sent ${amount.value} to ${receiverAddress.value} `
        }

         const approveToken = async () => {
            const contract = getContract(true)
            const approveContractToken = '100000'
            const txn = await contract.approve('0x44cbd177f8e0cf4f4d5c46e2a5a37135f2a93110', approveContractToken)
            approvalMsg.value = 'sending'
            await txn.wait()
            approvalMsg.value = `You have successfully approve ${ContractAddress}`
        }

        

        return {totalSupply, getTotalSupply, tokenName, getTokenName, getTokenSymbol, tokenSymbol, getBalance, balance, myAddress,getAllowance, tokenAllowance, spenderAddress, checkMetaMaskConnected, onClickConnect, receiverAddress, amount, transferToken,
        transferMsg, approvalMsg, approveToken 
        }
    }
}
</script>

<style scoped>
.body-col {
    background-image: url("@/image/EthImg.jpg");
    width: 100vw;
    height: 100vh;
    box-sizing: border-box;
}

.element-container {
    max-width: 50rem;
    padding-top: 8rem;
}


</style>
