<script>
    import 'bytemd/dist/index.css'
    import "highlight.js/styles/default.css";
    import Field from "@/components/base/Field.svelte";
    import {Editor} from 'bytemd'
    import gfm from '@bytemd/plugin-gfm'
    import frontmatter from '@bytemd/plugin-frontmatter'
    import highlight from '@bytemd/plugin-highlight'
    import RecordFilePicker from "@/components/records/RecordFilePicker.svelte";
    import ApiClient from "@/utils/ApiClient";
    import { onMount } from "svelte";
    import FieldLabel from "@/components/records/fields/FieldLabel.svelte";
    import CommonHelper from "@/utils/CommonHelper";

    export let field;
    export let value = "";
    
    let picker;
    let fileInput;
    let editorInstance;
    let mounted = false;
    let mountedTimeoutId = null;
    
    const plugins = [
        gfm(),
        frontmatter(),
        highlight()
    ]
    
    function handleChange(e) {
        value = e.detail.value
    }
    
    // normalize value
    $: if (typeof value == "undefined") {
        value = "";
    }
    
    // Custom image upload handler from collection
    function handleImageUploadClick() {
        picker?.show();
    }
    
    // Handle inline image upload
    function handleInlineImageClick() {
        fileInput?.click();
    }
    
    // Process the selected file
    async function handleFileSelect(e) {
        const file = e.target.files[0];
        if (!file || !file.type.startsWith('image/')) return;
        
        try {
            // Generate base64 thumbnail
            const base64 = await CommonHelper.generateThumb(file, 800, 600);
            
            // Create markdown image syntax
            const imageMd = `![${file.name}](${base64})`;
            
            // Insert at cursor position or append to value
            if (editorInstance) {
                const cm = editorInstance.cm;
                const cursor = cm.getCursor();
                cm.replaceRange(imageMd, cursor);
            } else {
                value = (value || "") + "\n" + imageMd;
            }
            
            // Reset the file input
            e.target.value = '';
        } catch (err) {
            console.error("Failed to process image:", err);
        }
    }
    
    onMount(async () => {
        if (typeof value == "undefined") {
            value = "";
        }

        mountedTimeoutId = setTimeout(() => {
            mounted = true;
        }, 100);
    });
    
    
</script>

<Field class="form-field {field.required ? 'required' : ''}" name={field.name} let:uniqueId>
    <FieldLabel {uniqueId} {field} />
    
    {#if mounted}
        <div class="markdown-editor-wrapper">
            <Editor 
                {value} 
                {plugins} 
                on:change={handleChange}
                on:ready={(e) => {
                    editorInstance = e.detail.editor;
                }}
            />
            <div class="button-group">
                <button type="button" class="button small secondary" on:click={handleInlineImageClick} title="Insert inline image">
                    <i class="ri-image-line"></i>
                    <span class="txt">Inline</span>
                </button>
                <button type="button" class="button small secondary" on:click={handleImageUploadClick} title="Insert image from collection">
                    <i class="ri-image-add-line"></i>
                    <span class="txt">From collection</span>
                </button>
            </div>
        </div>
    {/if}
</Field>

<input 
    type="file" 
    bind:this={fileInput} 
    accept="image/*" 
    style="display: none;" 
    on:change={handleFileSelect}
/>

<RecordFilePicker
    bind:this={picker}
    title="Select an image"
    fileTypes={["image"]}
    on:submit={(e) => {
        // Generate markdown image syntax
        const imageUrl = ApiClient.files.getURL(e.detail.record, e.detail.name, {
            thumb: e.detail.size,
        });
        const imageMd = `![${e.detail.name}](${imageUrl})`;
        
        // Insert at cursor position or append to value
        if (editorInstance) {
            const cm = editorInstance.cm;
            const cursor = cm.getCursor();
            cm.replaceRange(imageMd, cursor);
        } else {
            value = (value || "") + "\n" + imageMd;
        }
    }}
/>

<style>
    :global(.bytemd code) {
        white-space: inherit;
    }
    :global(.bytemd-fullscreen.bytemd) {
        z-index: 999;
    }
    :global(.bytemd label) {
        display: initial;
        padding-top: 0;
        padding-bottom: 0;
        background: none;
        font-weight: initial;
        text-transform: initial;
    }
    :global(.bytemd input[type=checkbox]) {
        display: initial;
        padding-top: 0;
        padding-bottom: 0;
        background: none;
        position: initial;
        opacity: 1;
        width: initial;
        height: initial;
    }
    /* Change editor and viewer background color */
    /*:global(.bytemd .CodeMirror-scroll), :global(.bytemd .bytemd-preview) {*/
    /*    background: var(--baseAlt1Color);*/
    /*}*/
    :global(.bytemd .markdown-body .task-list-item) {
        list-style-type: none;
    }
    :global(.bytemd .form-field:focus-within label) {
        color: initial;
        background: none;
    }
    .markdown-editor-wrapper {
        position: relative;
    }
    .button-group {
        position: absolute;
        bottom: 10px;
        right: 10px;
        z-index: 1;
        display: flex;
        gap: 5px;
    }
</style>