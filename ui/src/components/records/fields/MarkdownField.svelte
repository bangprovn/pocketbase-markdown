<script>
    import "bytemd/dist/index.css";
    import "highlight.js/styles/default.css";
    import Field from "@/components/base/Field.svelte";
    import { Editor } from "bytemd";
    import gfm from "@bytemd/plugin-gfm";
    import frontmatter from "@bytemd/plugin-frontmatter";
    import highlight from "@bytemd/plugin-highlight";
    import RecordFilePicker from "@/components/records/RecordFilePicker.svelte";
    import ApiClient from "@/utils/ApiClient";
    import { onMount, tick } from "svelte";
    import FieldLabel from "@/components/records/fields/FieldLabel.svelte";
    import CommonHelper from "@/utils/CommonHelper";

    export let field;
    export let value = "";

    let picker;
    let fileInput;
    let editorInstance;
    let mounted = false;
    let mountedTimeoutId = null;
    let editorContainer;
    let buttonGroup;

    const plugins = [gfm(), frontmatter(), highlight()];

    function handleChange(e) {
        value = e.detail.value;
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
        if (!file || !file.type.startsWith("image/")) return;

        try {
            // Generate base64 thumbnail
            const base64 = await CommonHelper.generateThumb(file, 800, 600);

            // Create markdown image syntax
            const imageMd = `![${file.name}](${base64})`;

            // Insert at cursor position
            insertAtCursor(imageMd);

            // Reset the file input
            e.target.value = "";
        } catch (err) {
            console.error("Failed to process image:", err);
        }
    }

    // Function to get the CodeMirror instance from the DOM
    function getCodeMirrorInstance() {
        if (!editorContainer) return null;

        // ByteMD creates a CodeMirror editor inside the container
        const cmWrapper = editorContainer.querySelector(".CodeMirror");
        if (cmWrapper && cmWrapper.CodeMirror) {
            return cmWrapper.CodeMirror;
        }
        return null;
    }

    // Function to add buttons to ByteMD toolbar
    function addButtonsToToolbar() {
        if (!buttonGroup || !editorContainer) return;

        const toolbar = editorContainer.querySelector(".bytemd-toolbar-right");
        if (toolbar) {
            toolbar.appendChild(buttonGroup);
        }
    }

    // Function to insert content at cursor position
    function insertAtCursor(content) {
        // Try to get CodeMirror instance directly from the DOM
        const cm = getCodeMirrorInstance();
        if (cm) {
            // Get current cursor position
            const cursor = cm.getCursor();
            // Insert content at cursor
            cm.replaceRange(content, cursor);
            // Focus the editor
            cm.focus();
        } else if (editorInstance && editorInstance.codemirror) {
            // Fallback to stored editorInstance if available
            const cursor = editorInstance.codemirror.getCursor();
            editorInstance.codemirror.replaceRange(content, cursor);
            editorInstance.codemirror.focus();
        } else {
            // Fallback to appending if CodeMirror not available
            value = (value || "") + "\n" + content;
        }
    }

    onMount(async () => {
        if (typeof value == "undefined") {
            value = "";
        }

        mountedTimeoutId = setTimeout(async () => {
            mounted = true;

            // Wait for the editor to be fully rendered
            await tick();
            await new Promise((resolve) => setTimeout(resolve, 500));

            // Try to get the CodeMirror instance after mounting
            const cm = getCodeMirrorInstance();
            if (cm) {
                editorInstance = { codemirror: cm };
            }

            // Add buttons to toolbar
            addButtonsToToolbar();
        }, 100);

        return () => {
            clearTimeout(mountedTimeoutId);
        };
    });
</script>

<Field class="form-field {field.required ? 'required' : ''}" name={field.name} let:uniqueId>
    <FieldLabel {uniqueId} {field} />

    {#if mounted}
        <div class="markdown-editor-wrapper" bind:this={editorContainer}>
            <Editor {value} {plugins} on:change={handleChange} />
            <div class="button-group" bind:this={buttonGroup}>
                <button
                    type="button"
                    class="button small secondary"
                    on:click={handleInlineImageClick}
                    title="Insert inline image"
                >
                    <i class="ri-image-line"></i>
                    <span class="txt">Inline</span>
                </button>
                <button
                    type="button"
                    class="button small secondary"
                    on:click={handleImageUploadClick}
                    title="Insert image from collection"
                >
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

        // Insert at cursor position
        insertAtCursor(imageMd);
    }}
/>

<style>
    :global(.bytemd code) {
        white-space: inherit;
    }
    :global(.bytemd-fullscreen.bytemd) {
        z-index: 20;
    }
    :global(.bytemd label) {
        display: initial;
        padding-top: 0;
        padding-bottom: 0;
        background: none;
        font-weight: initial;
        text-transform: initial;
    }
    :global(.bytemd input[type="checkbox"]) {
        display: initial;
        padding-top: 0;
        padding-bottom: 0;
        background: none;
        position: initial;
        opacity: 1;
        width: initial;
        height: initial;
    }
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
    
    /* Styling for buttons in toolbar */
    .button-group {
        display: flex;
        gap: 5px;
        margin-left: 5px;
    }
    
    /* Make toolbar buttons compact */
    .button-group .button {
        height: 26px;
        min-height: 26px;
        padding: 2px 8px;
        font-size: 12px;
    }
    
    .button-group .button .txt {
        display: none; /* Hide text in toolbar, only show icons */
    }
    
    .button-group .button i {
        margin: 0;
    }
</style>
