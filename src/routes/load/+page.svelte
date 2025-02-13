<script lang="ts">
    import MyHighlightsIcon from '$lib/icons/MyHighlights.svelte';
    import GlobalIcon from '$lib/icons/Global.svelte';
    import FollowsIcon from '$lib/icons/Follows.svelte';
	import RadioButton from '$lib/components/buttons/radio.svelte';
	import Highlight from '$lib/components/HighlightList.svelte';
    import { page } from '$app/stores';
    import { fetchArticle } from '$lib/article';
    import ArticleInterface from '$lib/interfaces/article';
    import HighlightInterface from '$lib/interfaces/highlights';
    import NoteInterface from '$lib/interfaces/notes';
    import { onMount } from 'svelte';
    import { Tooltip } from 'flowbite-svelte';
    import Widget from '../../Widget.svelte';
    import Avatar from '$lib/components/Avatar.svelte';
    import Name from '$lib/components/Name.svelte';
    import Article from '$lib/components/Article.svelte';
    import { NDKUser } from '@nostr-dev-kit/ndk';

    let url = $page.url.searchParams.get('url') || '';
    let author = $page.url.searchParams.get('author') || '';

    let mode = 'global';

    // function setMode() {
    //     switch (mode) {
    //         case 'my':
    //             myHighlights();
    //             break;
    //         case 'global':
    //             globalHighlights();
    //             break;
    //         case 'network':
    //             alert('coming soon™️!');
    //             break;
    //     }
    // }

    let articles;
    let article: any;
    let content: string;
    let authorHexpubkey: string;
    let highlights;
    let _highlights: App.Highlight[] = [];

    let notes;
    let _notes: App.Note[] = [];
    let activeSub: NDKSubscription | undefined;
    let fetchError: string | undefined;

    async function loadUrl() {
        articles = ArticleInterface.load({url});

        try {
            article = await fetchArticle(url);
        } catch (error: any) {
            fetchError = error.message;
        }

        content = article.content;
    }

    onMount(async () => {
        loadUrl();

        setTimeout(() => {
            highlights = HighlightInterface.load({url});
            activeSub = HighlightInterface.startStream({url});
            notes = NoteInterface.load({pubkeys: ['asas']});
        }, 1000);
    });

    $: {
        _highlights = ($highlights || []) as App.Highlight[];
        _notes = ($notes || []) as App.Note[];

        if (author && authorHexpubkey === undefined) {
            try {
                if (author.startsWith('npub')) {
                    authorHexpubkey = (new NDKUser({npub: author})).hexpubkey();
                } else {
                    authorHexpubkey = author;
                }
            } catch(e) {}
        }

        if (_highlights && content) {
            for (const highlight of _highlights) {
                content = content.replace(highlight.content, `<mark data-highlight-id="${highlight.id}">${highlight.content}</mark>`);
            }
        }
    }
</script>

<div class="flex flex-col sm:flex-row w-screen sm:gap-12">
    <div class="sm:w-3/5 text-lg p-8 bg-black shadow-lg text-justify text-slate-700 leading-loose flex flex-col gap-2">
        {#if article}
            <!-- Title -->
            <h1 class="text-3xl sm:text-4xl lg:text-6xl font-thin leading-normal font-serif text-white text-left">{article.title}</h1>
            {#if article.excerpt}
                <h2 class="text-xl truncate whitespace-nowrap sm:text-2xl lg:text-3xl font-thin leading-normal text-zinc-400 text-left mb-2">{article.excerpt}</h2>
            {/if}

            <!-- Author / URL -->
            {#if authorHexpubkey}
                <h2 class="flex flex-row items-center text-sm sm:text-sm gap-4">
                    <div class="flex flex-row gap-4 items-start">
                        <Avatar pubkey={authorHexpubkey} klass="h-8" />
                        <div class=" text-gray-500 text-lg">
                            <Name pubkey={authorHexpubkey} />
                        </div>
                    </div>
                </h2>
            {:else}

                <button class="text-orange-800 flex flex-row" on:click={() => { author = prompt(`Enter the author's npub`) }}>
                    Do you know the author's npub?
                </button>
            {/if}

            <div class="flex flex-row justify-between">
                {#if article.byline}
                    {article.byline}
                {:else if url}
                    <div class="text-slate-600 text-xs whitespace-nowrap">
                        {url}
                    </div>
                {:else}
                    <div></div>
                {/if}

                <!-- Publisher -->
                {#if article?.publisher && article?.publisher !== author}
                    <h2 class="flex flex-row items-center text-sm sm:text-sm gap-4">
                        Published by
                        <div class="flex flex-row items-center gap-2">
                            <Avatar pubkey={article.publisher} klass="h-10" />
                            <Name pubkey={article.publisher} />
                        </div>
                    </h2>
                {/if}
            </div>


            <!-- Highlight count on mobile -->
            {#if $highlights && $highlights.length > 0}
                <a href="#highlights" class="
                    sm:hidden
                    font-sans text-base
                    text-purple-500
                ">{$highlights?.length} highlights</a>
            {/if}

            <!-- Content -->
            <article class="my-6 font-serif">
                <Article>
                    {@html content}
                </Article>
            </article>
        {:else}
            {fetchError}
        {/if}
    </div>

    <!-- Sidebar -->
    <div class="sm:w-2/5 p-8 pt-4 sm:h-screen sm:overflow-scroll sm:fixed right-0">
        <div class="flex flex-row items-center justify-between mb-8">
            <div>
                <a href="/" class="
                    text-zinc-400 hover:text-white
                    font-semibold
                " name="highlights">
                    ⚡️ <span class="font-black">ZAP</span>WORTHY
                </a>
            </div>

            <div class="flex flex-row text-slate-300 items-center justify-center
                text-xs sm:text-lg
            ">
                <RadioButton bind:group={mode} value="my">
                    <MyHighlightsIcon />
                </RadioButton>
                <Tooltip placement="bottom">My Highlights</Tooltip>

                <RadioButton bind:group={mode} value="global">
                    <GlobalIcon />
                </RadioButton>
                <Tooltip placement="bottom">Global Highlights Feed</Tooltip>

                <RadioButton bind:group={mode} value="network">
                    <FollowsIcon />
                </RadioButton>
                <Tooltip placement="bottom">Highlights from people you follow</Tooltip>
            </div>
        </div>

        {#if _highlights}
            <div class="flex flex-col gap-6">
                {#each _highlights as highlight}
                    <Highlight {highlight} skipUrl={true} skipTitle={true} />
                {/each}
            </div>
        {/if}
    </div>
</div>

<Widget {url} loadHighlights={false} position="bottom-5 left-5 flex-col-reverse" explicitAuthorHexpubkey={authorHexpubkey} />
