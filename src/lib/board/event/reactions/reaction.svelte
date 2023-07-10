<script>
import { onMount, createEventDispatcher } from 'svelte'
import { page } from '$app/stores';
import { store } from '$lib/store/store.js'
import tippy from 'tippy.js';

$: state = $store?.states[$page?.params?.space]
$: sender_id = $store.credentials?.matrix_user_id
$: isOwner = state?.owner?.user_id === sender_id
$: isSpaceAdmin = $store?.power_levels?.space?.[$store?.credentials?.matrix_user_id] == 100

$: authenticated = $store?.authenticated && 
    $store?.credentials != null
    $store?.credentials?.access_token?.length > 0

const dispatch = createEventDispatcher()

export let event;
export let reaction;
export let isReply;

export let isTag;

$: sender= $store.credentials?.matrix_user_id

$: reacted = reaction?.senders?.findIndex(s => s === sender) > -1

$: notOP = isTag && 
    reaction?.senders?.[0] != sender_id

function reactToEvent() {
    if(isTag && !authenticated) {
        return
    }
    if(isTag && (!isSpaceAdmin && !isOwner)) {
        return
    }

    if(!authenticated) {
        store.startAuthenticating("login")
        return
    }

    if(notOP) {
        return
    }

    dispatch('react', reaction.key)
}

$: tagKey = reaction?.key?.split(':')[1]

let el;
let content;

let menu;

$: if(reaction && el) {
    menu = tippy(el, {
        content: content,
        placement: 'top',
        arrow: true,
        duration: 1,
        theme: 'inline',
    });
}

function localpart(id) {
    return id.split(':')[0].substring(1)
}
</script>

{#if !isTag}

<div class="tip fl-co" bind:this={content}>
    {#each reaction.senders as sender, i}
        <div class="sen" class:mb2={i != (reaction?.senders?.length - 1)}>
            @{localpart(sender)}
        </div>
    {/each}
</div>

<div class="reaction fl mr1 grd-c" 
    class:tag={isTag}
    bind:this={el}
    on:click|stopPropagation={reactToEvent}
    class:reacted={reacted}>
    <div class="emoji">
        {reaction.key}
    </div>
    <div class="t ml1">
        {reaction?.senders?.length}
    </div>
</div>
{/if}

{#if isTag}
<div class="tag fl mr1 grd" 
    class:nopointer={notOP}
    on:click|stopPropagation={reactToEvent}>

    <div class="ti grd-c">
        {tagKey}
    </div>
</div>
{/if}


<style>
.reaction {
    font-size: small;
    color: var(--text-light);
    background-color: var(--shade-2);
    border: 1px solid var(--shade-2);
    border-radius: 5px;
    padding-left: 0.25rem;
    padding-right: 0.25rem;
    padding-top: 0;
    padding-bottom: 0;
    height: 20px;
    cursor: pointer;
}

.tag{
}

.ti {
    font-size: small;
    background-color: var(--primary);
    color: white;
    font-weight: 500;
    padding-left: 0.25rem;
    padding-right: 0.25rem;
    height: 18px;
    border-radius: 3px;
}

.nopointer {
    cursor: default;
}

.reacted {
    border: 1px solid var(--primary);
}

.bg {
    background-color: var(--reaction-bg);
}

:global(:root) {
    --reaction-bg: #272727;
    --reaction-border: #272727;
}

:global(:root.light) {
    --reaction-bg: #f5f5f5;
    --reaction-border: #f0f0f0;
}

.emoji {
    font-size: small;
    font-weight: 500;
    line-height: 1.6;
}
.t {
    font-size: small;
    color: var(--text-light);
    line-height: 1.6;
}

.sen {
    font-size: small;
    font-weight: 500;
}

@media screen and (max-width: 1020px) {
    .reaction {
        max-height: 16px;
    }
    .emoji {
        font-size: 10px;
    }
    .t {
        font-size: 10px;
    }
}
</style>