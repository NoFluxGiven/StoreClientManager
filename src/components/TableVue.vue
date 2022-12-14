<template>
    <v-data-table :headers="headers" :items="items" sort-by="key" class="elevation-1" :loading="loading">
        <template v-slot:top>
            <div class="text-center">
                <v-snackbar v-model="snackbar" :timeout="timeout">
                    {{ snackbar_msg }}

                    <template v-slot:action="{ attrs }">
                        <v-btn color="blue" text v-bind="attrs" @click="snackbar = false">
                            Close
                        </v-btn>
                    </template>
                </v-snackbar>
            </div>
            <v-toolbar flat>
                <v-toolbar-title>Store Client Manager</v-toolbar-title>
                <v-divider class="mx-4" inset vertical></v-divider>
                <v-btn small class="mr-2" @click="createStore()" v-if="storeUUID===''">
                    <v-icon>
                        mdi-plus
                    </v-icon>
                    Create New Store
                </v-btn>
                <v-icon medium class="mr-2" @click="loadStore(storeUUID)" v-if="storeUUID!==''">
                    mdi-refresh
                </v-icon>
                <v-divider class="mx-4" inset vertical></v-divider>
                <v-dialog v-model="dialogStore" max-width="500px">
                    <template v-slot:activator="{ on, attrs }">
                        <v-btn small :color="storeUUID==='' ? 'warning' : 'success'" :loading="loading" class="mr-1"
                            v-bind="attrs" v-on="on">
                            {{ storeTitle }}
                        </v-btn>
                    </template>
                    <v-card>
                        <v-card-title>
                            <span class="text-h5">Load Store from UUID</span>
                        </v-card-title>

                        <v-card-text>
                            <v-container>
                                <v-row>
                                    <v-col cols="12" sm="9" md="11">
                                        <!-- <v-select v-model="bookmarkSelect"
                                            :hint="`${bookmarkSelect.val}, ${bookmarkSelect.key}`"
                                            :items="bookmarksList" item-text="val" item-name="val" item-value="key"
                                            label="Load Bookmark" @change="editedStoreUUID(key)" return-object
                                            single-line :disabled="editedStoreUUID!==''">
                                        </v-select> -->
                                        <v-combobox v-model="editedStoreUUID" :items="bookmarksList" item-text="val"
                                            item-name="val" item-value="key" outlined label="UUID" persistent-hint
                                            hint="Paste or Generate a UUIDv4 compliant Store UUID, or load one from your Bookmarks."
                                            :return-object="false">
                                        </v-combobox>

                                    </v-col>
                                </v-row>
                            </v-container>
                        </v-card-text>

                        <v-card-actions>
                            <v-spacer></v-spacer>
                            <v-btn color="green darken-1" text @click="generateUUID()">
                                Generate
                            </v-btn>
                            <v-btn color="red darken-1" text @click="closeStore">
                                Cancel
                            </v-btn>
                            <v-btn color="blue darken-1" text @click="saveStore">
                                {{storeUUID === "" ? "Load" : "Change"}}
                            </v-btn>

                        </v-card-actions>
                    </v-card>
                </v-dialog>
                <v-divider class="mx-4" inset vertical></v-divider>
                <v-dialog v-model="dialogBookmark" max-width="500px" v-if="storeUUID!==''">
                    <template v-slot:activator="{ on, attrs }">
                        <v-btn small class="mr-1" v-on="on" text v-bind="attrs">
                            <v-icon>{{bookmarkTitle != '' ? "mdi-bookmark" : "mdi-bookmark-plus"}}</v-icon>
                            <v-icon small v-if="bookmarkTitle != ''">mdi-pencil</v-icon>
                        </v-btn>
                    </template>
                    <v-card>
                        <v-card-title>
                            <span class="text-h5">Bookmark</span>
                        </v-card-title>

                        <v-card-text>
                            <v-container>
                                <v-row>
                                    <v-col cols="12" sm="9" md="11">
                                        <v-text-field v-model="bookmarkName"
                                            :label="bookmarkTitle !== '' ? 'Rename' : 'Add Bookmark'"
                                            :placeholder="bookmarkTitle"
                                            hint="A descriptive name for the loaded Store.">
                                        </v-text-field>
                                    </v-col>
                                </v-row>
                            </v-container>
                        </v-card-text>

                        <v-card-actions>
                            <v-spacer></v-spacer>
                            <v-btn color="blue darken-1" text @click="closeBookmark">
                                Cancel
                            </v-btn>
                            <v-btn :color="bookmarkName !== '' ? 'green darken-1' : 'warning darken-1'" text
                                @click="saveBookmark">
                                {{bookmarkTitle !== '' ? "Rename Bookmark" : "Add Bookmark"}}
                            </v-btn>

                        </v-card-actions>
                    </v-card>

                </v-dialog>
                <v-divider class="mx-4" inset vertical></v-divider>
                <v-col>
                    <v-row>
                        <span class="black--text" v-if="bookmarkTitle!==''">{{bookmarkTitle}}</span>
                    </v-row>
                    <v-row>
                        <span class="grey--text" @click="copyStoreUUID()">{{storeUUID}} </span>
                    </v-row>
                </v-col>
                <!-- <v-icon medium class="mr-6" @click="copyStoreUUID()" v-if="storeUUID!==''"> -->
                <!-- mdi-clipboard -->
                <!-- </v-icon> -->
                <v-spacer></v-spacer>
                <v-dialog v-model="dialog" max-width="500px">
                    <template v-slot:activator="{ on, attrs }">
                        <v-btn color="primary" class="mb-2" v-bind="attrs" v-on="on" :disabled="storeUUID===''">
                            New Item
                        </v-btn>
                    </template>
                    <v-card>
                        <v-card-title>
                            <span class="text-h5">{{ formTitle }}</span>
                        </v-card-title>

                        <v-card-text>
                            <v-container>
                                <v-row>
                                    <v-col cols="12" sm="9" md="8">
                                        <v-text-field v-model="editedItem.key" label="Key"
                                            hint="The key to assign the value to.">
                                        </v-text-field>
                                    </v-col>
                                    <v-col cols="12" sm="9" md="8">
                                        <v-text-field v-model="editedItem.val" label="Value"
                                            hint="The value itself, which will be stored as a string.">
                                        </v-text-field>
                                    </v-col>
                                </v-row>
                            </v-container>
                        </v-card-text>

                        <v-card-actions>
                            <v-spacer></v-spacer>
                            <v-btn color="blue darken-1" text @click="close">
                                Cancel
                            </v-btn>
                            <v-btn color="blue darken-1" text @click="save">
                                Save
                            </v-btn>
                        </v-card-actions>
                    </v-card>
                </v-dialog>
                <v-dialog v-model="dialogDelete" max-width="500px">
                    <v-card>
                        <v-card-title class="text-h5">Are you sure you want to delete this item?</v-card-title>
                        <v-card-actions>
                            <v-spacer></v-spacer>
                            <v-btn color="blue darken-1" text @click="closeDelete">Cancel</v-btn>
                            <v-btn color="blue darken-1" text @click="deleteItemConfirm">OK</v-btn>
                            <v-spacer></v-spacer>
                        </v-card-actions>
                    </v-card>
                </v-dialog>
            </v-toolbar>
        </template>
        <template v-slot:[`item.val`]="{ item }">
            {{item.val}}
        </template>
        <template v-slot:[`item.actions`]="{ item }">
            <v-icon small class="mr-2" @click="editItem(item)">
                mdi-pencil
            </v-icon>
            <v-icon small @click="deleteItem(item)">
                mdi-delete
            </v-icon>
        </template>
    </v-data-table>
</template>
  
<script>

import axios from 'axios'

export default {
    data: () => ({
        expanded: [],
        dialog: false,
        dialogDelete: false,
        dialogStore: false,
        dialogBookmark: false,
        recent: "",
        bookmarks: {},
        bookmarksList: [],
        bookmarkName: "",
        bookmarkSelect: {},
        loading: false,
        snackbar: false,
        snackbar_msg: "",
        timeout: 6000,
        headers: [
            {
                text: 'Key',
                align: 'start',
                sortable: true,
                value: 'key',
            },
            {
                text: 'Value',
                sortable: true,
                value: 'val',
            },
            { text: 'Actions', value: 'actions', sortable: false },
        ],
        items: [],
        storeUUID: "",
        editedStoreUUID: "",
        editedIndex: -1,
        editedItem: {
            key: "",
            val: ""
        },
        defaultItem: {
            key: "",
            val: ""
        },
    }),

    computed: {
        formTitle() {
            return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
        },
        storeTitle() {
            // let b = this.bookmarks;
            // let name = b[this.storeUUID] || "< Untitled >";

            return this.storeUUID === "" ? "Load Store" : `Change Store `
        },
        bookmarkTitle() {
            let b = this.bookmarks;
            let name = b[this.storeUUID] || "";

            return name;
        }
    },

    watch: {
        dialog(val) {
            val || this.close()
        },
        dialogDelete(val) {
            val || this.closeDelete()
        },
        dialogStore(val) {
            val || this.closeStore()
        },
        dialogBookmark(val) {
            val || this.closeBookmark()
        }
    },

    created() {
        this.initialize()
    },

    methods: {
        initialize() {
            this.items = [
            ]

            this.updateBookmarks();

            this.recent = this.bookmarks["@recent"];

            if (this.recent) {
                this.loadStore(this.recent)
            }
        },

        editItem(item) {
            this.editedIndex = this.items.indexOf(item)
            this.editedItem = Object.assign({}, item)
            this.dialog = true
        },

        deleteItem(item) {
            this.editedIndex = this.items.indexOf(item)
            this.editedItem = Object.assign({}, item)
            this.dialogDelete = true
        },

        async deleteItemConfirm() {
            this.closeDelete()
            await this.deleteStore(this.storeUUID, this.items[this.editedIndex]);
            await this.loadStore(this.storeUUID);
            // this.items.splice(this.editedIndex, 1)

        },

        close() {
            this.dialog = false
            this.$nextTick(() => {
                this.editedItem = Object.assign({}, this.defaultItem)
                this.editedIndex = -1
            })
        },

        closeDelete() {
            this.dialogDelete = false
            this.$nextTick(() => {
                this.editedItem = Object.assign({}, this.defaultItem)
                this.editedIndex = -1
            })
        },

        closeStore() {
            this.dialogStore = false
            this.$nextTick(() => {
                this.editedStoreUUID = "";
            })
        },

        closeBookmark() {
            this.dialogBookmark = false
            this.$nextTick(() => {
                this.bookmarkName = "";
            })
        },

        async save() {
            this.close()

            const success = await this.pushStore(this.storeUUID, this.editedItem);
            if (!success)
                return

            await this.loadStore(this.storeUUID);
            // if (this.editedIndex > -1) {
            //     Object.assign(this.items[this.editedIndex], this.editedItem)
            // } else {
            //     this.items.push(this.editedItem);
            // }
        },

        async saveStore() {
            await this.loadStore(this.editedStoreUUID);


            this.closeStore();
        },

        saveBookmark() {

            if (this.bookmarkName === '' && this.bookmarkSelect) {
                this.loadStore(this.bookmarkSelect.key);
                this.closeBookmark();
                return true;
            }

            this.bookmarkStoreUUID(this.storeUUID, this.bookmarkName);

            this.updateBookmarks();

            this.closeBookmark();
        },

        async loadStore(uuid) {
            if (!uuid) {
                this.snackbar = true;
                this.snackbar_msg = "Provide a UUID to load a Store.";
                return false;
            }
            console.log(`Load store ${uuid}...`)
            let data;
            this.loading = true;
            try {
                const res = await axios.get(`https://store.zapier.com/api/records?secret=${uuid}`)
                data = res.data;
            }
            catch (err) {
                console.log(`Load failed\n${err.response.data.message}...`)
                this.snackbar = true;
                this.snackbar_msg = `Store load error: ${err.response.data.error}`;
                this.loading = false;
                return false;
            }

            this.loading = false;

            console.log(`Load complete.`)

            this.storeUUID = uuid;
            this.updateRecent();

            this.items = [];

            // convert to required schema

            for (var [k, v] of Object.entries(data)) {
                this.items.push({
                    key: k,
                    val: v
                });
            }

            return true;
        },

        generateUUID() {
            let newUUID = crypto.randomUUID();
            this.editedStoreUUID = newUUID;
        },

        async createStore() {
            let newUUID = crypto.randomUUID();

            this.storeUUID = newUUID;

            console.log(`Generated ${newUUID}`)

            await this.loadStore(newUUID);

            console.log(`Loaded ${newUUID}`)
        },

        async pushStore(uuid, { key, val }) {
            // console.log(`${val === undefined ? 'Delete from' : 'Push to'} store ${uuid}...`)
            // if (!val && val !== undefined) {
            //     this.snackbar = true;
            //     this.snackbar_msg = "Enter a valid value.";
            //     return false;
            // }

            this.loading = true;
            try {
                await axios.post(`https://store.zapier.com/api/records?secret=${uuid}`, { [key]: val })
            }
            catch (err) {
                console.log(`Push failed\n${err.response.data.message}...`)
                this.snackbar = true;
                this.snackbar_msg = `Item push error: ${err.response.data.error}`;
                this.loading = false;
                return false;
            }

            this.loading = false;

            return true;
        },

        async deleteStore(uuid, { key }) {
            console.log(`Deleting ${key}`)

            this.loading = true;

            try {
                await axios.patch(`https://store.zapier.com/api/records?secret=${uuid}`, { action: "delete", data: [key] })
            }
            catch (err) {
                console.log(`Delete failed\n${err.response.data.message}...`)
                this.snackbar = true;
                this.snackbar_msg = `Item delete error: ${err.response.data.error}`;
                this.loading = false;
                return false;
            }

            this.loading = false;
        },

        copyStoreUUID() {
            navigator.clipboard.writeText(this.storeUUID);
            this.snackbar = true;
            this.snackbar_msg = "UUID copied.";
        },

        bookmarkStoreUUID(uuid, name) {
            localStorage.setItem(uuid, name)
            this.bookmarks = { ...localStorage }
        },

        updateBookmarks() {
            this.bookmarks = { ...localStorage };

            this.updateBookmarksList();
        },

        updateRecent() {
            this.recent = this.storeUUID;
            localStorage.setItem("@recent", this.recent);
        },

        updateBookmarksList() {

            let bookmarkList = [];

            for (var [k, v] of Object.entries(this.bookmarks)) {
                if (k === "@recent" || k === "") continue;
                bookmarkList.push({
                    key: k,
                    val: v
                })
            }

            console.log(bookmarkList);

            this.bookmarksList = bookmarkList;
        }
    },
}
</script>