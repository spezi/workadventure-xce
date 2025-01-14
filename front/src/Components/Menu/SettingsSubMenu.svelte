<script lang="typescript">
    import { localUserStore } from "../../Connexion/LocalUserStore";
    import { isSilentStore, videoConstraintStore } from "../../Stores/MediaStore";
    import { audioManagerFileStore, audioManagerVisibilityStore } from "../../Stores/AudioManagerStore";
    import { requestedCameraState } from "../../Stores/MediaStore";
    import { HtmlUtils } from "../../WebRtc/HtmlUtils";
    import { isMobile } from "../../Enum/EnvironmentVariable";
    import { gameManager } from "../../Phaser/Game/GameManager";

    let fullscreen: boolean = localUserStore.getFullscreen();
    let notification: boolean = localUserStore.getNotification() === "granted";
    let blockAudio: boolean = localUserStore.getBlockAudio();
    let forceCowebsiteTrigger: boolean = localUserStore.getForceCowebsiteTrigger();
    let ignoreFollowRequests: boolean = localUserStore.getIgnoreFollowRequests();
    let alwaysSilent: boolean = localUserStore.getAlwaysSilent();
    let noVideo: boolean = localUserStore.getNoVideo();
    let disableAnimations: boolean = localUserStore.getDisableAnimations();
    let gameQuality: number = localUserStore.getGameQualityValue();
    let videoQuality: number = localUserStore.getVideoQualityValue();
    let blockExternalContent: boolean = localUserStore.getBlockExternalContent();
    let previousGameQuality = gameQuality;
    let previousVideoQuality = videoQuality;
    let oldBlockExternalContent = blockExternalContent;

    function saveSettings() {
        let reload = false;

        if (gameQuality !== previousGameQuality) {
            previousGameQuality = gameQuality;
            localUserStore.setGameQualityValue(gameQuality);
            reload = true;
        }

        if (videoQuality !== previousVideoQuality) {
            previousVideoQuality = videoQuality;
            videoConstraintStore.setFrameRate(videoQuality);
            reload = true;
        }

        if (blockExternalContent !== oldBlockExternalContent) {
            oldBlockExternalContent = blockExternalContent;
            localUserStore.setBlockExternalContent(blockExternalContent);
            reload = true;
        }

        if (reload) {
            window.location.reload();
        }
    }

    function changeFullscreen() {
        const body = HtmlUtils.querySelectorOrFail("body");
        if (body) {
            if (document.fullscreenElement !== null && !fullscreen) {
                document.exitFullscreen();
            } else {
                body.requestFullscreen();
            }
            localUserStore.setFullscreen(fullscreen);
        }
    }

    function changeNotification() {
        if (Notification.permission === "granted") {
            localUserStore.setNotification(notification ? "granted" : "denied");
        } else {
            Notification.requestPermission().then((response) => {
                if (response === "granted") {
                    localUserStore.setNotification(notification ? "granted" : "denied");
                } else {
                    localUserStore.setNotification("denied");
                    notification = false;
                }
            });
        }
    }

    function changeBlockAudio() {
        if (blockAudio) {
            audioManagerFileStore.unloadAudio();
            audioManagerVisibilityStore.set(false);
        }
        localUserStore.setBlockAudio(blockAudio);
    }

    function changeForceCowebsiteTrigger() {
        localUserStore.setForceCowebsiteTrigger(forceCowebsiteTrigger);
    }

    function changeIgnoreFollowRequests() {
        localUserStore.setIgnoreFollowRequests(ignoreFollowRequests);
    }

    function changeAlwaysSilent() {
        localUserStore.setAlwaysSilent(alwaysSilent);
        const scene = gameManager.getCurrentGameScene();
        const silentZone = scene.isSilentZone();
        const silent = alwaysSilent || silentZone;
        scene.connection?.setSilent(silent);
        // We need to make sure this flag toggles at least once to update UI
        isSilentStore.set(!silent);
        isSilentStore.set(silent);
    }

    function changeNoVideo() {
        localUserStore.setNoVideo(noVideo);
        if (noVideo) {
            requestedCameraState.disableWebcam();
        }
    }

    function changeDisableAnimations() {
        localUserStore.setDisableAnimations(disableAnimations);
        if (disableAnimations) {
            gameManager.getCurrentGameScene().animatedTiles.pause();
        } else {
            gameManager.getCurrentGameScene().animatedTiles.resume();
        }
    }
</script>

<div class="settings-main" on:submit|preventDefault={saveSettings}>
    <section>
        <h3>Game framerate</h3>
        <div class="nes-select is-dark">
            <select bind:value={gameQuality}>
                <option value={120}>{isMobile() ? "High (120 fps)" : "High quality (120 fps)"}</option>
                <option value={60}>{isMobile() ? "Medium (60 fps)" : "Medium quality (60 fps, recommended)"}</option>
                <option value={40}>{isMobile() ? "Minimum (40 fps)" : "Minimum quality (40 fps)"}</option>
                <option value={20}>{isMobile() ? "Small (20 fps)" : "Small quality (20 fps)"}</option>
            </select>
        </div>
    </section>
    <section>
        <h3>Video framerate</h3>
        <div class="nes-select is-dark">
            <select bind:value={videoQuality}>
                <option value={30}>{isMobile() ? "High (30 fps)" : "High video quality (30 fps)"}</option>
                <option value={20}
                    >{isMobile() ? "Medium (20 fps)" : "Medium video quality (20 fps, recommended)"}</option
                >
                <option value={10}>{isMobile() ? "Minimum (10 fps)" : "Minimum video quality (10 fps)"}</option>
                <option value={5}>{isMobile() ? "Small (5 fps)" : "Small video quality (5 fps)"}</option>
            </select>
        </div>
    </section>
    <section>
        <label>
            <input type="checkbox" class="nes-checkbox is-dark" bind:checked={blockExternalContent} />
            <span>Block external content (tabs, split-screens and embedded websites such as video streams)</span>
        </label>
    </section>
    <section class="settings-section-save">
        <p>Note: Changing these settings will restart the game.</p>
        <button type="button" class="nes-btn is-primary" on:click|preventDefault={saveSettings}>Save</button>
    </section>
    <section class="settings-section-noSaveOption">
        <label>
            <input
                type="checkbox"
                class="nes-checkbox is-dark"
                bind:checked={fullscreen}
                on:change={changeFullscreen}
            />
            <span>Fullscreen</span>
        </label>
        <label>
            <input
                type="checkbox"
                class="nes-checkbox is-dark"
                bind:checked={notification}
                on:change={changeNotification}
            />
            <span>Notifications</span>
        </label>
        <label>
            <input
                type="checkbox"
                class="nes-checkbox is-dark"
                bind:checked={blockAudio}
                on:change={changeBlockAudio}
            />
            <span>Block ambient sounds and music</span>
        </label>
        <label>
            <input
                type="checkbox"
                class="nes-checkbox is-dark"
                bind:checked={forceCowebsiteTrigger}
                on:change={changeForceCowebsiteTrigger}
            />
            <span>Always ask before opening websites and Jitsi Meet rooms</span>
        </label>
        <label>
            <input
                type="checkbox"
                class="nes-checkbox is-dark"
                bind:checked={ignoreFollowRequests}
                on:change={changeIgnoreFollowRequests}
            />
            <span>Ignore requests to follow other users</span>
        </label>
        <label>
            <input
                type="checkbox"
                class="nes-checkbox is-dark"
                bind:checked={alwaysSilent}
                on:change={changeAlwaysSilent}
            />
            <span>Silent mode (disable proximity chat)</span>
        </label>
        <label>
            <input type="checkbox" class="nes-checkbox is-dark" bind:checked={noVideo} on:change={changeNoVideo} />
            <span>Disable camera</span>
        </label>
        <label>
            <input
                type="checkbox"
                class="nes-checkbox is-dark"
                bind:checked={disableAnimations}
                on:change={changeDisableAnimations}
            />
            <span>Disable tile animations</span>
        </label>
    </section>
</div>

<style lang="scss">
    div.settings-main {
        height: calc(100% - 40px);
        overflow-y: auto;

        section {
            width: 100%;
            padding: 20px 20px 0;
            margin-bottom: 20px;
            text-align: center;

            div.nes-select select:focus {
                outline: none;
            }
        }
        section.settings-section-save {
            text-align: center;
            p {
                margin: 16px 0;
            }
        }
        section.settings-section-noSaveOption {
            display: grid;
            align-items: center;
            flex-wrap: wrap;

            label {
                flex: 1 1 auto;
                text-align: left;
                margin: 0 0 15px;
            }
        }
    }

    @media only screen and (max-width: 800px), only screen and (max-height: 800px) {
        div.settings-main {
            section {
                padding: 0;
            }
        }
    }
</style>
