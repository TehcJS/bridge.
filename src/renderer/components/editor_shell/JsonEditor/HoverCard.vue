<template>
    <v-menu
        v-model="is_visible"
        :close-on-content-click="false"
        :position-x="x_position"
        :position-y="y_position"
    >
        <v-card>
            <v-card-text>
                <pre>{{ data.slice(0, 200) + (data.length > 200 ? "..." : "") }}</pre>
                
            </v-card-text>
            <v-divider/>

            <v-text-field
                solo
                v-model="current_comment"
                @input="updateComment"
                label="Click to add a comment"
                prepend-inner-icon="mdi-pencil"
                hide-details
                style="margin: 4px 0px;"
            />
            <v-divider/>

            <v-card-actions>
                <template
                    v-for="(btn, i) in buttons"
                >
                    <v-spacer v-if="btn === 'space'" :key="i"/>
                    <v-tooltip
                        v-else
                        :key="i"
                        bottom
                        :color="btn.color || 'primary'"
                        :style="`margin-right: ${ i + 1 !== buttons.length ? 4 : 0}px;`"
                    >
                        <v-btn
                            slot="activator"
                            :color="btn.color || 'primary'" 
                            round
                            icon
                            @click="btn.action"
                        >
                            <v-icon color="white">{{ btn.icon }}</v-icon>
                        </v-btn>
                        <span>{{ btn.title }}</span>
                    </v-tooltip>
                </template>
            </v-card-actions>
        </v-card>
    </v-menu>
</template>

<script>
import TabSystem from "../../../scripts/TabSystem";
import { clipboard } from "electron";
import { JSONAction } from "../../../scripts/TabSystem/CommonHistory";
import EventBus from "../../../scripts/EventBus";
import { DOC_WINDOW } from "../../../scripts/documentation/main";
import NodeShortcuts from "../../../scripts/editor/Shortcuts";

export default {
    name: "json-editor-hover-card",
    data() {
        return {
            current_comment: "",
            buttons: [
                {
                    title: "Documentation",
                    icon: "mdi-book-open-page-variant",
                    color: "orange",
                    action: () => {
                        this.is_visible = false;
                        
                        DOC_WINDOW.open("entities");
                        let e = document.getElementById(TabSystem.getCurrentNavContent());
                        window.setTimeout(() => { if(e) e.scrollIntoView() }, 1000);
                    }
                },
                "space",
                {
                    title: "Move Down",
                    icon: "mdi-chevron-down",
                    action: () => TabSystem.moveCurrentDown()
                },
                {
                    title: "Move Up",
                    icon: "mdi-chevron-up",
                    action: () => TabSystem.moveCurrentUp()
                },
                {
                    title: "Copy",
                    icon: "mdi-content-copy",
                    color: "success",
                    action: () => {
                        this.is_visible = false;

                        NodeShortcuts.copy();
                    }
                },
                {
                    title: "Cut",
                    icon: "mdi-content-cut",
                    color: "success",
                    action: () => {
                        this.is_visible = false;

                        NodeShortcuts.cut();
                    }
                },
                {
                    title: "Paste",
                    icon: "mdi-download",
                    color: "success",
                    action: () => {
                        this.is_visible = false;

                        NodeShortcuts.paste();
                    }
                },
                {
                    title: "Delete",
                    icon: "mdi-delete",
                    color: "error",
                    action: () => {
                        this.is_visible = false;
                        TabSystem.deleteCurrent();
                    }
                }
            ]
        }
    },
    computed: {
        data() {
            return this.$store.state.EditorHover.data;
        },
        is_visible: {
            get() {
                this.updateCurrentComment();
                return this.$store.state.EditorHover.is_visible;
            },
            set() {
                this.$store.commit("hideEditorHoverCard");
            }
        },
        x_position() {
            return this.$store.state.EditorHover.x_position;
        },
        y_position() {
            return this.$store.state.EditorHover.y_position;
        }
    },
    methods: {
        updateComment(val) {
            TabSystem.getCurrentNavObj().comment = val;
        },
        updateCurrentComment() {
            this.$nextTick(() => {
                try { this.current_comment = TabSystem.getCurrentNavObj().comment; } catch(e) { /*console.log(e)*/ }
            });
        }
    }
}
</script>

<style scoped>
    pre {
        font-family: 'Roboto', sans-serif !important;
    }
</style>
