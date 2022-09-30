<template>
    <div v-if="!isLoggedIn && networkMetamask">
        <button @click="connectWallet()" v-if="isMetamaskSupported || checkIfConnected()">Connect to metamask</button>
        <h2 v-else>Install Metamask Extension</h2>
        <h3 v-if="ready === false">{{errorMessage}}</h3>
    </div>
    <div v-if="isLoggedIn">
        <p v-if="checkIfConnected()">We are connected : {{account}}</p>
        <button @click="disconnectWallet()">Disconnect</button>
    </div>
    <div v-if="!networkMetamask">
        <button disabled>Connect to metamask</button>
        <p>You are on the wrong network</p>
    </div>
</template>

<script setup lang="ts">
import {onMounted, ref} from '@vue/runtime-core'
import { useTimeout } from '@vueuse/core'

const isMetamaskSupported = ref(false); // Check if the user have metamask and if it's supported
const isLoggedIn = ref(false); // Check if the user is log
const account = ref(); // Address of the user
const errorMessage = ref(); // We are doing error here
const networkMetamask = ref(false); // Check if the user are on the good network
const { ready, start } = useTimeout(2500, { controls: true })

onMounted(async() => {
    isMetamaskSupported.value = typeof(window as any).ethereum !== 'undefined';

    if (checkIfConnected() === true)
        isLoggedIn.value = true;

    // console.log('CONNECTED :', (window as any).ethereum.isConnected());

    const chainId = await (window as any).ethereum.request({ method: 'eth_chainId' });
    if (chainId === process.env.VUE_APP_CHAIN_ID) {
        networkMetamask.value = true;
    }
});

const checkIfConnected = () => {
    if (localStorage.hasOwnProperty('address')) {
        account.value = localStorage.address
        return true;
    }
    return false;
}

(window as any).ethereum.on('accountsChanged', (accounts:any) => {
    if (localStorage.hasOwnProperty('address')) {
        const user = accounts[0];
        if (!user) {
            isLoggedIn.value = false;
            errorMessage.value = "You have disconnected, please reconnect."
            start();
            localStorage.removeItem('address')

        }
        else {
            localStorage.setItem('address', user);
            account.value = localStorage.address;
        }
    }
});

(window as any).ethereum.on('chainChanged', (chainId:string) => { // when you change the network
    if (localStorage.hasOwnProperty('address')) {
        localStorage.removeItem('address')
    }
    // Handle the new chain.
    // Correctly handling chain changes can be complicated.
    // We recommend reloading the page unless you have good reason not to.
    window.location.reload();
});

const connectWallet = async() => { // when the user try to connect
    await (window as any).ethereum
        .request({ method: 'eth_requestAccounts' })
        .then((response:any) => { // if the user connect
            isLoggedIn.value = true;
            const account = response[0];
            localStorage.setItem('address', account);
        })
        .catch((err:any) => { // if the user cancel the connexion
            start();
            errorMessage.value = "You have cancelled your connexion."
        });
};

const disconnectWallet = async() => { // when the user disconnect his metamask
    isLoggedIn.value = false;
    localStorage.removeItem('address');
    account.value = undefined;
}

</script>

<style scoped>

</style>