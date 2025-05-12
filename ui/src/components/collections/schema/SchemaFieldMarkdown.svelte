<script>
    import Field from "@/components/base/Field.svelte";
    import SchemaField from "@/components/collections/schema/SchemaField.svelte";

    export let field;
    export let key = "";
</script>

<SchemaField bind:field {key} on:rename on:remove on:duplicate {...$$restProps}>
    <svelte:fragment slot="options">
        <Field class="form-field m-b-sm" name="fields.{key}.maxSize" let:uniqueId>
            <label for={uniqueId}>Max size <small>(bytes)</small></label>
            <input
                type="number"
                id={uniqueId}
                step="1"
                min="0"
                max={Number.MAX_SAFE_INTEGER}
                value={field.maxSize || ""}
                on:input={(e) => (field.maxSize = parseInt(e.target.value, 10))}
                placeholder="Default to max ~5MB"
            />
        </Field>
    </svelte:fragment>
</SchemaField> 