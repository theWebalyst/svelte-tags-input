<script>
import { createEventDispatcher } from 'svelte';

const dispatch = createEventDispatcher();
let tags = [];
let tag;
let arrelementsmatch = [];

export let setTags;
export let addKeys;
export let maxTags;
export let onlyUnique;
export let removeKeys;
export let placeholder;
export let allowPaste;
export let allowDrop;
export let splitWith;
export let autoComplete;

$: setTags = setTags || [];
$: addKeys = addKeys || false;
$: maxTags = maxTags || false;
$: onlyUnique = onlyUnique || false;
$: removeKeys = removeKeys || false;
$: placeholder = placeholder || "";
$: allowPaste = allowPaste || false;
$: allowDrop = allowDrop || false;
$: splitWith = splitWith || ",";
$: autoComplete = autoComplete || false;

$: {
    tags = [...setTags];
    setTags = [];
}

function setTag(event) {
    
    const currentTag = event.target.value;

    if (event.keyCode === 13) {
        addTag(currentTag);
    }
    
    if (event.keyCode === 8 && tag == "") {
        tags.pop();  
        tags = tags;

        dispatch('tags', {
            tags: tags
        });
    }
    
    if (addKeys) {
        addKeys.forEach(function(key) {
            if (key === event.keyCode) {
                switch (event.keyCode) {
                case 188:
                    // Remove comma if keycode to add tag is comma
                    addTag(currentTag.substring(0, currentTag.length - 1));
                    break;                    
                case 9:
                    event.preventDefault();
                    addTag(currentTag);
                    break;
                default:
                    addTag(currentTag);
                    break;
                }
            }
        });
    }
    
    if (removeKeys) {
        removeKeys.forEach(function(key) {
            if (key === event.keyCode) {
                tags.pop();  
                tags = tags;
                tag = "";

                dispatch('tags', {
                    tags: tags
                });
            }
        });
    }

}

function addTag(currentTag) {

    currentTag = currentTag.trim();

    if (currentTag == "") return;
    if (maxTags && tags.length == maxTags) return;
    
    if(onlyUnique) {
        if (tags.includes(currentTag)) return;
    }
    
    tags.push(currentTag)
    tags = tags;
    tag = "";

    dispatch('tags', {
		tags: tags
	});

}

function removeTag(i) {
    
    tags.splice(i, 1);
    tags = tags;

    dispatch('tags', {
		tags: tags
	});

}

function onPaste(e){

    if(!allowPaste) return;

    e.preventDefault();

    const data = getClipboardData(e);
    const tags = splitTags(data).map(tag => addTag(tag));
    
}

function onDrop(e){

    if(!allowDrop) return;

    e.preventDefault();

    const data = e.dataTransfer.getData("Text");
    const tags = splitTags(data).map(tag => addTag(tag));

}

function getClipboardData(e) {

    if (window.clipboardData) {
        return window.clipboardData.getData('Text')
    }

    if (e.clipboardData) {
        return e.clipboardData.getData('text/plain')
    }

    return ''
}

function splitTags(data) {
    return data.split(splitWith).map(d => d.trim());    
}

function getMatchElements(e) {

    if(!autoComplete) return;
    
    var x = e.target.value;
    
    if (x == "") {
        arrelementsmatch = [];
        return;
    }

    var matchs = autoComplete.filter(e =>e.toLowerCase().includes(x.toLowerCase()));

    arrelementsmatch = matchs;
}

</script>

<div class="svelte-tags-input-layout">
    {#if tags.length > 0}
        {#each tags as tag, i}
            <span class="svelte-tags-input-tag">{tag} <span class="svelte-tags-input-tag-remove" on:click={() => removeTag(i)}> ×</span></span>
        {/each}
    {/if}
    <input type="text" bind:value={tag} on:keydown={setTag} on:keyup={getMatchElements} on:paste={onPaste} on:drop={onDrop} class="svelte-tags-input" placeholder={placeholder}>
    
</div>

{#if autoComplete && arrelementsmatch.length > 0}
    <ul class="svelte-tags-input-matchs">
        {#each arrelementsmatch as element, i}
            <li value="element" on:click={() => addTag(element)}>{element}</li>
        {/each}
    </ul>
{/if}

<style>
/* main */

.svelte-tags-input,
.svelte-tags-input-tag {
    font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,Oxygen,Ubuntu,Cantarell,"Fira Sans","Droid Sans","Helvetica Neue",sans-serif;
    font-size: 14px;
    padding: 2px 5px;
}

/* svelte-tags-input-layout */

.svelte-tags-input-layout {
    display:-webkit-box;
    display:-ms-flexbox;
    display:flex;
    -ms-flex-wrap:wrap;
        flex-wrap:wrap;
    -webkit-box-align:center;
        -ms-flex-align:center;
            align-items:center;
    padding: 0px 5px 5px 5px;
    border: solid 1px #CCC;
    background: #FFF;
    border-radius: 2px;
}

.svelte-tags-input-layout:focus,
.svelte-tags-input-layout:hover {
    border: solid 1px #000;    
}

/* svelte-tags-input */

.svelte-tags-input {
    -webkit-box-flex: 1;
        -ms-flex: 1;
            flex: 1; 
    margin: 0;
    margin-top: 5px;
    border:none;
}

.svelte-tags-input:focus {
    outline:0;
}

/* svelte-tags-input-tag */

.svelte-tags-input-tag {
    display:-webkit-box;
    display:-ms-flexbox;
    display:flex;
    white-space: nowrap;
    list-style:none;
    background: #000;
    color: #FFF;
    border-radius: 2px;
    margin-right: 5px;
    margin-top: 5px;
}

.svelte-tags-input-tag:hover {
    /*background: #CCC;*/
}

.svelte-tags-input-tag-remove {
    cursor:pointer;
}

/* svelte-tags-input-matchs */

.svelte-tags-input-matchs {
    margin:3px 0;
    padding: 0px;
    border: solid 1px #CCC;
    border-radius: 2px;
    max-height:310px;
    overflow:scroll;
    overflow-x:hidden;
}

.svelte-tags-input-matchs li {
    list-style:none;
    padding:5px;
    border-radius: 2px;
    cursor:pointer;
}

.svelte-tags-input-matchs li:hover {
    background:#000;
    color:#FFF;
}
</style>