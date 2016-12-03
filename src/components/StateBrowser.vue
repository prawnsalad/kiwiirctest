<template>
    <div class="kiwi-statebrowser">
        <div class="kiwi-statebrowser-scrollarea">
            <div class="kiwi-statebrowser-networks">
                <div class="kiwi-statebrowser-network" v-for="network in networks">
                    <a class="kiwi-statebrowser-network-name" @click="setActiveBuffer(network.serverBuffer())">{{network.name}}</a>

                    <div class="kiwi-statebrowser-channels">
                        <div
                            v-for="buffer in orderedBuffers(network.buffers)"
                            class="kiwi-statebrowser-channel"
                            v-bind:class="{
                                'kiwi-statebrowser-channel-active': isActiveBuffer(buffer),
                                'kiwi-statebrowser-channel-notjoined': buffer.isChannel() && !buffer.joined
                            }"
                        >
                            <div class="kiwi-statebrowser-channel-name" @click="setActiveBuffer(buffer)">{{buffer.name}}</div>
                            <div class="kiwi-statebrowser-channel-labels">
                                <transition name="kiwi-statebarowser-channel-label-transition">
                                <div v-if="buffer.flags.unread" class="kiwi-statebrowser-channel-label">
                                    {{buffer.flags.unread}}
                                </div>
                                </transition>
                            </div>

                            <div
                                class="kiwi-statebrowser-channel-settings"
                                @click.stop="showBufferPopup(buffer)"
                            >:</div>

                            <div
                                v-if="shouldShowPopup(buffer)"
                                class="kiwi-statebrowser-channel-popup"
                                @click.stop=""
                            >
                                <buffer-settings v-bind:buffer="buffer"></buffer-settings>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <div class="kiwi-statebrowser-options">
                <a @click="clickAddNetwork">Add network</a>
                <a>Settings</a>
                <a>Forget Me</a>
            </div>
        </div>
    </div>
</template>

<script>

import _ from 'lodash';
import state from 'src/libs/state';
import NetworkSettings from './NetworkSettings';
import BufferSettings from './BufferSettings';

export default {
    data: function data() {
        return {
            // Name of the buffer that should show its popup
            popup_buffername: null,
            popup_networkid: null,
        };
    },
    props: ['networks'],
    components: {
        BufferSettings,
    },
    methods: {
        setActiveBuffer: function switchContainer(buffer) {
            // Clear any active component to show the buffer again
            state.$emit('active.component', null);
            state.setActiveBuffer(buffer.networkid, buffer.name);
        },
        isActiveBuffer: function isActiveBuffer(buffer) {
            return (
                buffer.networkid === state.ui.active_network &&
                buffer.name === state.ui.active_buffer
            );
        },
        orderedBuffers: function orderedBuffers(buffers) {
            // Since vuejs will sort in-place and update views when .sort is called
            // on an array, clone it first so that we have a plain array to sort
            let list = buffers.map(b => b);

            list = _.filter(list, buffer => !buffer.isServer());
            return list.sort((a, b) => a.name.localeCompare(b.name));
        },
        showNetworkSettings: function showNetworkSettings(network) {
            state.$emit('active.component', NetworkSettings, {
                network,
            });
        },
        showBufferPopup: function showBufferPopup(buffer) {
            if (!buffer) {
                this.popup_buffername = null;
                this.popup_networkid = null;
            } else {
                this.popup_buffername = buffer.name;
                this.popup_networkid = buffer.networkid;
            }
        },
        shouldShowPopup: function shouldShowPopup(buffer) {
            if (
                this.popup_networkid !== buffer.networkid ||
                this.popup_buffername !== buffer.name
            ) {
                return false;
            }

            // Make sure this buffer exists (it may have been deleted since)
            if (!state.getBufferByName(buffer.networkid, buffer.name)) {
                return false;
            }

            return true;
        },
        clickAddNetwork: function clickAddNetwork() {
            let nick = 'Guest' + Math.floor(Math.random() * 100);
            let network = state.addNetwork('New Network', nick, {});
            state.$emit('active.component', NetworkSettings, {
                network,
            });
        },
    },
    computed: {
        activeBuffer: function activeBuffer() {
            return {
                networkid: state.ui.active_network,
                buffer: state.ui.active_buffer,
            };
        },
    },
    created: function created() {
        state.$on('document.clicked', () => {
            this.showBufferPopup(null);
        });
    },
};
</script>

<style>
.kiwi-statebrowser {
    box-sizing: border-box;
}

.kiwi-statebrowser-scrollarea {
    overflow: auto;
    height: 100%;
    width: 100%;
}
.kiwi-statebrowser-networks {
    padding-bottom: 60px; /* .kiwi-statebrowser-options height+padding */
}
.kiwi-statebrowser-channel {
    position: relative;
    display: flex;
}
.kiwi-statebrowser-channel-name {
    flex: 1;
}

.kiwi-statebrowser-channel-settings {
    display: none;
}
.kiwi-statebrowser-channel:hover .kiwi-statebrowser-channel-settings {
    display: inline-block;
}
.kiwi-statebrowser-channel-popup {
    display: block;
    position: absolute;
    left: 100%;
    width: 100%;
}

.kiwi-statebrowser-options {
    position: absolute;
    bottom: 0;
    padding: 15px;
    height: 30px;
}
</style>