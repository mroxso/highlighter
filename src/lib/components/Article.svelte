<div class="
    text-xl sm:p-8 bg-zinc-900 sm:shadow-2xl text-justify text-gray-300 leading-loose
    overflow-scroll
">
    <slot />
</div>

<style>
    :global(article h1) {
        font-weight: 600;
        font-size: 2rem;
        margin-top: 1.5rem;
    }

    :global(article h2) {
        font-weight: 600;
        font-size: 1.75rem;
        margin-top: 1.5rem;
    }

    :global(article h3) {
        font-weight: 600;
        font-size: 1.5rem;
        margin-top: 1.5rem;
    }

    :global(article p) {
        margin-bottom: 1rem;
    }

    :global(article blockquote) {
        font-family: var(--article-heading);
        font-size: 21px;
        overflow-wrap: break-word;
        padding: 0 0 0 50px;
        margin: 1.5rem 0;
    }

    :global(article ul) {
        list-style: disc;
        margin-left: 1.5rem;
    }

    :global(article a) {
        color:blueviolet;
    }

    :global(article img) {
        max-height: 420px !important;
        max-width: 100% !important;
        margin: 1rem auto;
    }

    :global(article mark) {
        background-color: rgb(227, 151, 35);
        color: black;
        padding: 0.25rem;
    }
</style>