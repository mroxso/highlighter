<script lang="ts">
    import UserInterface from '$lib/interfaces/users';
    import type { Observable } from 'dexie';

    export let pubkey: string | undefined = undefined;
    export let userProfile: App.UserProfile | undefined = undefined;

    let prevPubkey: string | undefined = undefined;

    let defaultName = `[${pubkey?.slice(0, 5)}]`;;

    let observeUserProfile: Observable<App.UserProfile> | undefined = undefined;
    let name: string | undefined = userProfile?.displayName;

    $: {
        if (pubkey !== prevPubkey && !userProfile) {
            prevPubkey = pubkey;
            observeUserProfile = UserInterface.get({ hexpubkey: pubkey });
        }

        if (observeUserProfile && $observeUserProfile) {
            userProfile = ($observeUserProfile||{}) as App.UserProfile;
        }

        defaultName = `[${pubkey?.slice(0, 5)}]`;
        name = userProfile?.displayName || defaultName;
    }
</script>

{name}