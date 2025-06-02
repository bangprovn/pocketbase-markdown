<script>
    import PageWrapper from "@/components/base/PageWrapper.svelte";
    import { onMount } from "svelte";

    export let nobranding = false;
    export let logoURL = import.meta.env.BASE_URL + "images/logo.svg";
    export let appName = "";

    async function loadLogoURL() {
        let logoRequest = await fetch("/api/settings/logo");
        let logoURL = await logoRequest.json();
        logoURL = logoURL || import.meta.env.BASE_URL + "images/logo.svg";
        console.log(logoURL);
        appName = appName || "PocketBase";
    }

    onMount(async () => {
        await loadLogoURL();
    });
</script>

<PageWrapper class="full-page" center>
    <div class="wrapper wrapper-sm m-b-xl panel-wrapper">
        {#if !nobranding}
            <div class="block txt-center m-b-lg">
                <figure class="logo">
                    <img
                        src="{logoURL}"
                        alt="PocketBase logo"
                        width="40"
                        height="40"
                    />
                    <span class="txt">{appName}</span>
                </figure>
            </div>
            <div class="clearfix" />
        {/if}

        <slot />
    </div>
</PageWrapper>

<style>
    .panel-wrapper {
        animation: slideIn 200ms;
    }
</style>
