<script>
    import { onMount } from "svelte";
    // import QR from 'svelte-qr';
    import { ndk } from './lib/store';
    import { NDKNip07Signer } from "@nostr-dev-kit/ndk";

    let hasNostrNip07 = true;
    let publicKey = null;
    let nip46URI;
    let adapterConfig;
    let type = 'nip07'; //localStorage.getItem('nostrichat-type');

    onMount(() => {
        if (type === 'nip07') {
            useNip07();
        } else if (type === 'nip-46') {
            useNip46();
        }
    });

    function useNip07() {
        try {
            $ndk.signer = new NDKNip07Signer();
            $ndk.signer.user();
        } catch (e) {
            hasNostrNip07 = false;
            console.error(e);
        }
    }

    import { generatePrivateKey, getPublicKey } from 'nostr-tools';
    import { Connect, ConnectURI } from '@nostr-connect/connect';

    async function useDiscardableKeys() {
        // chatAdapter.set(new NstrAdapterDiscadableKeys(adapterConfig))
    }


    async function useNip46() {
        let key = localStorage.getItem('nostrichat-nostr-connect-key');
        let publicKey = localStorage.getItem('nostrichat-nostr-connect-public-key');

        if (key) {
            chatAdapter.set(new NstrAdapterNip46(publicKey, key, adapterConfig))
            return;
        }

        key = generatePrivateKey();

        const connect = new Connect({ secretKey: key, relay: 'wss://nostr.vulpem.com' });
        connect.events.on('connect', (connectedPubKey) => {
            localStorage.setItem('nostrichat-nostr-connect-key', key);
            localStorage.setItem('nostrichat-nostr-connect-public-key', connectedPubKey);
            localStorage.setItem('nostrichat-type', 'nip-46');

            console.log('connected to nostr connect relay')
            publicKey = connectedPubKey;
            chatAdapter.set(new NstrAdapterNip46(publicKey, key))
            nip46URI = null;
        });
        connect.events.on('disconnect', () => {
            console.log('disconnected from nostr connect relay')
        })
        await connect.init();

        let windowTitle, currentUrl, currentDomain;

        try {
            windowTitle = window.document.title || 'Nostrichat';
            currentUrl = new URL(window.location.href);
            currentDomain = currentUrl.hostname;
        } catch (e) {
            currentUrl = window.location.href;
            currentDomain = currentUrl;
        }

        const connectURI = new ConnectURI({
            target: getPublicKey(key),
            relay: 'wss://nostr.vulpem.com',
            metadata: {
                name: windowTitle,
                description: '🔉🔉🔉',
                url: currentUrl,
            },
        });

        nip46URI = connectURI.toString();
    }

    function Nip46Copy() {
        navigator.clipboard.writeText(nip46URI);
    }
</script>

<h1 class="font-bold text-xl mb-3">
    How would you like to connect?
</h1>

{#if publicKey}
    <p class="text-gray-400 mb-3 font-bold">
        Nostr Connect is a WIP, not fully implemented yet!
    </p>

    <p class="text-gray-400 mb-3">
        You are currently connected with the following public key:
        <span>{publicKey}</span>
    </p>
{/if}

{#if nip46URI}
    <p class="text-gray-600 mb-3">
        Scan this with your Nostr Connect (click to copy to clipboard)
    </p>

    <!-- <div class="bg-white w-full p-3"
        on:click|preventDefault={Nip46Copy}>
        <QR text={nip46URI} />
    </div> -->

    <button class="
        bg-purple-900
        hover:bg-purple-700
        w-full
        p-2
        rounded-xl
        text-center
        font-regular
        text-white
    " on:click|preventDefault={() => { nip46URI = null; }}>
        Cancel
    </button>
{:else if !publicKey}
    <div class="flex flex-col gap-1">
        {#if hasNostrNip07}
            <button class="
                bg-purple-900
                hover:bg-purple-700
                w-full
                p-4
                rounded-xl
                text-center
                font-regular
                text-gray-200
            " on:click|preventDefault={useNip07}>
                Browser Extension (NIP-07)
            </button>
        {/if}


        <button class="
            bg-purple-900
            hover:bg-purple-700
            w-full
            p-4
            rounded-xl
            text-center
            font-regular
            text-gray-200
        " on:click|preventDefault={useNip46}>
            Nostr Connect (NIP-46)
        </button>

        <button class="
            bg-purple-900
            hover:bg-purple-700
            w-full
            p-4
            rounded-xl
            text-center
            font-regular
            text-gray-200
        " on:click|preventDefault={useDiscardableKeys}>
            Anonymous
            <span class="text-xs text-gray-300">
                (Ephemeral Keys)
            </span>
        </button>
    </div>
{/if}

<style>
	@tailwind base;
	@tailwind components;
	@tailwind utilities;
</style>