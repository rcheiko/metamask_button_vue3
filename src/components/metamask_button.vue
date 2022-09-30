<template>
    <div v-if="!isLoggedIn && networkMetamask">
        <button @click="connectWallet()" v-if="isMetamaskSupported">Connect to metamask</button>
        <h2 v-else>Install Metamask Extension</h2>
        <h3 v-if="ready === false">{{errorMessage}}</h3>
    </div>
    <div v-if="isLoggedIn && networkMetamask">
        <p>We are connected : {{account}}</p>
        <button @click="disconnectWallet()">Disconnect</button>
    </div>
    <div v-if="!networkMetamask">
        <button disabled>Connect to metamask</button>
        <p>You are on the wrong network</p>
    </div>
</template>

<script setup lang="ts">
import {ref} from '@vue/runtime-core'
import { useTimeout } from '@vueuse/core'

const isMetamaskSupported = ref(false); // Check if the user have metamask and if it's supported
const isLoggedIn = ref(false); // Check if the user is log
const account = ref(); // Address of the user
const errorMessage = ref(); // We are doing error here
const networkMetamask = ref(false); // Check if the user are on the good network
const { ready, start } = useTimeout(2500, { controls: true });

(window as any).ethereum.on('connect', async(connectInfo: any) => {
    await new Promise(resolve => setTimeout(resolve, 0));
    isMetamaskSupported.value = typeof(window as any).ethereum !== 'undefined';

    if ((window as any).ethereum.selectedAddress) {
        isLoggedIn.value = true;
        account.value = (window as any).ethereum.selectedAddress;
    }

    if (connectInfo.chainId === process.env.VUE_APP_CHAIN_ID) {
        networkMetamask.value = true;
    }
});

(window as any).ethereum.on('accountsChanged', (accounts:any) => {
    const user = accounts[0];
    if (!user) {
        isLoggedIn.value = false;
        errorMessage.value = "You have disconnected, please reconnect."
        start();
        account.value = undefined;
    }
    else
        account.value = user;
});

(window as any).ethereum.on('chainChanged', (chainId:string) => { // when you change the network
    // Handle the new chain.
    // Correctly handling chain changes can be complicated.
    // We recommend reloading the page unless you have good reason not to.
    window.location.reload();
});

const connectWallet = async() => { // when the user try to connect
    await (window as any).ethereum
        .request({ method: 'eth_requestAccounts' })
        .then(() => { // if the user connect
            isLoggedIn.value = true;
            account.value = (window as any).ethereum.selectedAddress;
        })
        .catch(() => { // if the user cancel the connexion
            errorMessage.value = "You have cancelled your connexion."
            start();
        });
};

const disconnectWallet = async() => { // when the user disconnect his metamask
    isLoggedIn.value = false;
    account.value = undefined;
}

</script>

<style scoped>

</style>