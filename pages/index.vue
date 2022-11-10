<template>
<div class="">
<div class="container mt-5">
        <!-- <ValidationObserver> -->
        <button @click="getName">get name</button>
        {{name}}
         <form class="mb-5 mt-0" @submit.prevent="filterResult">

          <label class="mt-3">Wallet Address:</label>
          <!-- <ValidationProvider rules="" v-slot="{ errors }"> -->
         <b-form-input
         v-model="name"
         placeholder="Enter beneficiary wallet address"
           />
         <!-- <span class="" style="color:red">{{ errors[0] }}</span> -->
          <!-- </ValidationProvider>  -->

          <label class="mt-3">Duration:</label>
          <!-- <ValidationProvider rules="" v-slot="{ errors }"> -->
         <b-form-input
         v-model="duration"
           />
         <!-- <span class="" style="color:red">{{ errors[0] }}</span> -->
          <!-- </ValidationProvider>  -->

          <label class="mt-3">Amount:</label>
          <!-- <ValidationProvider rules="" v-slot="{ errors }"> -->
         <b-form-input
         v-model="amount"
           />
         <!-- <span class="" style="color:red">{{ errors[0] }}</span> -->
          <!-- </ValidationProvider>  -->

         
          <b-button
          @click="filterResult"
          class="btn-classes-active update-btn py-2"
          :disabled="crud || !name && !duration && !amount"
          size="sm"
          block
          >
          {{ 
             crud ? 'Sending': 'Submit'
           }}
          </b-button>
          
          </form>
          <!-- </ValidationObserver> -->
</div>
</div>
</template>

<script>
import {ref,onMounted, reactive } from '@nuxtjs/composition-api';
import bunzz from 'bunzz-sdk';
import { DAPP_ID, API_KEY } from "@/constant"
export default {
   setup(){
    const name = ref('') 
    const amount = ref('')
    const duration = ref('')
    const contract = ref('')
    const crud = ref('')
    const vestedSchd =reactive({
        _beneficiary : "",
        _start:"",
        _cliff:"",
        _duration:"",
        _slicePeriodSeconds:"",
      _revocable:"",
      __amount:"",




    })


    onMounted(()=>{
      initContract()
    })

 const handle = async () => {
            const allow = await bunzz.initWithConnect({
                dappId: DAPP_ID,
                apiKey: API_KEY,
                
})
return allow
        }

        const MODULE_NAME = "Vesting"
        const initContract = async () => {
      try {
        const handler = await handle();
        const getContract = await handler.getContract(MODULE_NAME);
        // console.log('ttt', contract)
   contract.value = getContract;
      } catch (error) {
        console.error(error);
      }
}

 const getName=async()=>{
  const resp =  await contract?.value?.owner()
  name.value= resp.data
 }

 const createScheduler=async()=>{
  
 }
 

    return {amount, duration, name, contract, crud , getName}
   }
}
</script>

<style scoped>



</style>
