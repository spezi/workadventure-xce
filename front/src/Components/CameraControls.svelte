<script lang="typescript">
    import { requestedScreenSharingState, screenSharingAvailableStore } from "../Stores/ScreenSharingStore";
    import { isSilentStore, requestedCameraState, requestedMicrophoneState } from "../Stores/MediaStore";
    import { localUserStore } from "../Connexion/LocalUserStore";
    import monitorImg from "./images/monitor.svg";
    import monitorCloseImg from "./images/monitor-close.svg";
    import cinemaImg from "./images/cinema.svg";
    import cinemaCloseImg from "./images/cinema-close.svg";
    import microphoneImg from "./images/microphone.svg";
    import microphoneCloseImg from "./images/microphone-close.svg";
    import layoutPresentationImg from "./images/layout-presentation.svg";
    import layoutChatImg from "./images/layout-chat.svg";
    import { layoutModeStore } from "../Stores/StreamableCollectionStore";
    import { LayoutMode } from "../WebRtc/LayoutManager";
    import { peerStore } from "../Stores/PeerStore";
    import { onDestroy } from "svelte";

    let camInstructionsVisible = false;

    function screenSharingClick(): void {
        if (isSilent) return;
        if ($requestedScreenSharingState === true) {
            requestedScreenSharingState.disableScreenSharing();
        } else {
            requestedScreenSharingState.enableScreenSharing();
        }
    }

    function cameraClick(): void {
        if (isSilent) return;
        if (localUserStore.getNoVideo()) {
            camInstructionsVisible = true;
            return;
        }
        if ($requestedCameraState === true) {
            requestedCameraState.disableWebcam();
        } else {
            requestedCameraState.enableWebcam();
        }
    }

    function microphoneClick(): void {
        if (isSilent) return;
        if ($requestedMicrophoneState === true) {
            requestedMicrophoneState.disableMicrophone();
        } else {
            requestedMicrophoneState.enableMicrophone();
        }
    }

    function switchLayoutMode() {
        if ($layoutModeStore === LayoutMode.Presentation) {
            $layoutModeStore = LayoutMode.VideoChat;
        } else {
            $layoutModeStore = LayoutMode.Presentation;
        }
    }

    function unblockCamera() {
        localUserStore.setNoVideo(false);
        requestedCameraState.enableWebcam();
        camInstructionsVisible = false;
    }

    let isSilent: boolean;
    const unsubscribeIsSilent = isSilentStore.subscribe((value) => {
        isSilent = value;
    });
    onDestroy(unsubscribeIsSilent);
</script>

<div>
    <div class="interact-menu nes-container is-rounded" hidden={!camInstructionsVisible}>
        <section class="interact-menu-question">
            <p>You have disabled your camera in game settings. Do you want to override this and enable your camera?</p>
        </section>
        <section class="interact-menu-action">
            <button type="button" class="nes-btn is-success" on:click|preventDefault={unblockCamera}>Yes</button>
            <button
                type="button"
                class="nes-btn is-error"
                on:click|preventDefault={() => (camInstructionsVisible = false)}>No</button
            >
        </section>
    </div>
    <div class="btn-cam-action">
        <div class="btn-layout" on:click={switchLayoutMode} class:hide={$peerStore.size === 0}>
            {#if $layoutModeStore === LayoutMode.Presentation}
                <img src={layoutPresentationImg} style="padding: 2px" alt="Switch to mosaic mode" />
            {:else}
                <img src={layoutChatImg} style="padding: 2px" alt="Switch to presentation mode" />
            {/if}
        </div>
        <div
            class="btn-monitor"
            on:click={screenSharingClick}
            class:hide={!$screenSharingAvailableStore || isSilent}
            class:enabled={$requestedScreenSharingState}
        >
            {#if $requestedScreenSharingState && !isSilent}
                <img src={monitorImg} alt="Start screen sharing" />
            {:else}
                <img src={monitorCloseImg} alt="Stop screen sharing" />
            {/if}
        </div>
        <div class="btn-video" on:click={cameraClick} class:disabled={!$requestedCameraState || isSilent}>
            {#if $requestedCameraState && !isSilent}
                <img src={cinemaImg} alt="Turn on webcam" />
            {:else}
                <img src={cinemaCloseImg} alt="Turn off webcam" />
            {/if}
        </div>
        <div class="btn-micro" on:click={microphoneClick} class:disabled={!$requestedMicrophoneState || isSilent}>
            {#if $requestedMicrophoneState && !isSilent}
                <img src={microphoneImg} alt="Turn on microphone" />
            {:else}
                <img src={microphoneCloseImg} alt="Turn off microphone" />
            {/if}
        </div>
    </div>
</div>

<style lang="scss">
    div.interact-menu {
        pointer-events: auto;
        user-select: none;
        background-color: #333333;
        color: whitesmoke;

        position: absolute;
        left: 20vw;
        width: 60vw;
        top: 60vh;
        margin: auto;

        section.interact-menu-question {
            margin: 4px;
            margin-bottom: 20px;

            p {
                font-size: 1.05em;
                font-weight: bold;
            }
        }

        section.interact-menu-action {
            display: grid;
            grid-gap: 10%;
            grid-template-columns: 45% 45%;
            margin-bottom: 20px;
            margin-left: 5%;
            margin-right: 5%;
        }
    }
</style>
