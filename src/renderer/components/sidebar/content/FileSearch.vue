<template>
    <div>
        <v-toolbar color="expanded_sidebar" flat height="30px">
            <v-tooltip color="tooltip" bottom>
                <template v-slot:activator="{ on }">
                    <v-btn icon text @click.stop="reset" v-on="on" class="toolbar-button" small>
                        <v-icon small>mdi-refresh</v-icon>
                    </v-btn>
                </template>
                <span>Refresh</span>
            </v-tooltip>

            <!-- <v-tooltip color="tooltip" bottom>
                <template v-slot:activator="{ on }">
                    <v-btn icon text @click.stop="search_all = !search_all" v-on="on" class="toolbar-button" small>
                        <v-icon v-if="!search_all" small>mdi-numeric-1-circle</v-icon>
                        <v-icon v-else small>mdi-alpha-a-circle</v-icon>
                    </v-btn>
                </template>
                <span v-if="!search_all">Search Current File</span>
                <span v-else>Search All Files</span>
            </v-tooltip> -->
        </v-toolbar>
        <v-divider/>

        <v-container>
            <v-text-field
                label="Search"
                append-outer-icon="mdi-magnify"
                hide-details
                solo
                dense
                v-model="search_val"
                @click:append-outer="search"
                @keydown.enter.native="search"
            />
            <v-text-field
                label="Replace"
                append-outer-icon="mdi-chevron-right"
                style="margin: 8px 0;"
                hide-details
                solo
                dense
                v-model="replace_val"
                @click:append-outer="replace"
                @keydown.enter.native="replace"
            />
        </v-container>

        <v-container class="px14-font">
            <v-divider/>

            <span v-if="selected_nodes_total > 0">Selected Nodes: {{ selected_nodes_total }}</span>
            <span v-else>Start searching for JSON nodes by opening a file and typing into the "Search" field.</span>

            <ul>
                <li
                    v-for="n in selected_nodes"
                    :key="n.uuid"
                >
                     <node-preview
                        :node_context="n"
                    />
                </li>
            </ul>
        </v-container>
    </div>
</template>

<script>
import TabSystem from '../../../scripts/TabSystem';
import JSONTree from '../../../scripts/editor/JsonTree';
import InformationWindow from '../../../scripts/commonWindows/Information';
import EventBus from '../../../scripts/EventBus';
import NodePreview from '../../common/NodePreview';

export default {
    name: "content-file-search",
    components: {
        NodePreview
    },
    data() {
        return {
            search_val: "",
            replace_val: "",
            search_all: false,
            selection: []
        }
    },
    mounted() {
        EventBus.on("updateTabUI", this.reset);
        EventBus.on("updateSelectedTab", this.reset);
    },
    destroyed() {
        EventBus.off("updateTabUI", this.reset);
        EventBus.off("updateSelectedTab", this.reset);
    },
    computed: {
        selected_nodes_total() {
            return this.selection.reduce((prev, curr) => prev + curr.nodes.length, 0);
        },
        selected_nodes() {
            return this.selection.reduce((prev, curr) => prev.concat(curr.nodes), []);
        }
    },
    methods: {
        search() {
            if(this.search_all) {

            } else {
                let sel = TabSystem.getSelected();
                if(!sel) return new InformationWindow("ERROR", "You need to open a file to get started with file search.");

                let c = sel.content;
                if(c instanceof JSONTree) {
                    let nodes = c.searchAll(this.search_val, true);
                    nodes.forEach(c => c.mark_color = "rgba(119, 119, 119, 0.3)");

                    this.selection.push({
                        search_val: this.search_val,
                        nodes
                    });
                    console.log(this.selection[this.selection.length - 1]);
                }
            }
        },
        replace() {
            if(this.selection.length === 0) return;

            this.selection.forEach(({ search_val, nodes }) => {
                nodes.forEach(n => {
                    if(n.key === search_val) n.editKey(this.replace_val, true);
                    else n.edit(this.replace_val, true);
                    console.log(n.key, n.data, search_val, n);
                });
            });
            TabSystem.setCurrentUnsaved();
            this.reset();
        },

        reset() {
            this.selection.forEach(({ nodes }) => nodes.forEach(n => n.mark_color = undefined));
            this.selection = [];
        }
    }
}
</script>

<style scoped>
    p {
        padding: 0.5em;
    }
</style>
