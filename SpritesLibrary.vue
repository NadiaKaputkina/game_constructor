<template>

    <v-card>
        <v-card-title
            class="headline grey lighten-2"
            primary-title>
            Библиотека спрайтов
        </v-card-title>

        <!--поиск-->
        <div class="sprites-wrap">

            <div class="search">
                <!--поиск по имени-->
                <v-combobox v-model="selectedNames"
                            :items="filteredNames"
                            chips
                            dense
                            clearable
                            label="Название"
                            multiple
                            hide-selected>
                    <template v-slot:selection="{ attrs, item }">
                        <v-chip v-bind="attrs"
                                label
                                close
                                @click:close="onRemove('selectedNames', item)">
                            {{ item }}
                        </v-chip>
                    </template>
                </v-combobox>

                <!--поиск по уровню-->
                <v-combobox v-model="selectedIds"
                            :items="filteredIds"
                            chips
                            dense
                            clearable
                            label="Уровень"
                            multiple
                            hide-selected>
                    <template v-slot:selection="{ attrs, item }">
                        <v-chip v-bind="attrs"
                                label
                                close
                                @click:close="onRemove('selectedIds', item)">
                            {{ item }}
                        </v-chip>
                    </template>
                </v-combobox>

                <!--поиск по тегам-->
                <v-combobox v-model="selectedTags"
                            :items="filteredTags"
                            chips
                            dense
                            clearable
                            label="Теги"
                            multiple
                            hide-selected>
                    <template v-slot:selection="{ attrs, item }">
                        <v-chip v-bind="attrs"
                                label
                                close
                                @click:close="onRemove('selectedTags', item)">
                            {{ item }}
                        </v-chip>
                    </template>
                </v-combobox>

                <v-card-text class="title">
                    Вы выбрали: {{nameSelectedSprite.name}}
                </v-card-text>

                <!--закрыть библиотеку-->
                <v-card-actions>
                    <v-btn color="primary"
                           @click="$emit('close-library-modal', nameSelectedSprite)"
                           :disabled="!isSpriteSelected">
                        Выбрать
                    </v-btn>
                </v-card-actions>
            </div>


            <div class="sprites">
                <!--иконки спрайтов-->
                <sprites-icon v-for="(icon, i) of filteredLibraryTextures"
                              @select-sprite="onSelectedSprite(icon, $event)"
                              :icon="icon"
                              :key="i"
                              :keywordsLibrary="keywordsLibrary">
                </sprites-icon>
            </div>
        </div>

    </v-card>

</template>

<script>

    import { ids} from '@/data/spritesTags';

    import SpritesIcon from "./SpritesIcon";

    export default {
        name: "SpritesLibrary",
        components: {
            'sprites-icon': SpritesIcon,
        },
        props: {
            libraryModal: Boolean,
            libraryTextures: Array,
            keywordsLibrary: Array,
        },
        data() {
            return {
                ids,

                selectedNames: [],
                selectedIds: [],
                selectedTags: [],

                isSpriteSelected: false,
                nameSelectedSprite: {name:''}
            }
        },

        watch: {
            selectedNames: function () {
                this.isSpriteSelected = false
            },
            selectedIds: function () {
                this.isSpriteSelected = false
            },
            selectedTags: function () {
                this.isSpriteSelected = false
            },
        },

        computed: {
            filteredTags: function () {
                return this.keywordsLibrary.map(tag => tag.name);
            },

            filteredNames: function () {
                return this.libraryTextures.map(item => item.name);
            },

            filteredIds: function () {
                return this.ids;
            },

            /**
             * Фильтруются спрайты согласно поиску
             */
            filteredLibraryTextures: function () {
                let filteredLibrary = [...this.libraryTextures];

                /*--поиск спрайтов по имени--*/
                if (this.selectedNames.length) {
                    filteredLibrary = filteredLibrary.filter(el =>
                        this.selectedNames.some(
                            name => el.name.includes(name)
                        )
                    );
                }

                /*--поиск спрайтов по id--*/
                if (this.selectedIds.length) {
                    filteredLibrary = filteredLibrary.filter(el =>
                        this.selectedIds.some(
                            id => {
                                let indexId = this.ids.indexOf(id);
                                return el.levelsId.includes(indexId)
                            }
                        )
                    );
                }

                /*--поиск спрайтов по тегам--*/
                if (this.selectedTags.length) {
                    filteredLibrary = filteredLibrary.filter(el =>
                        this.selectedTags.some(tagName => {

                            const index = this.keywordsLibrary.findIndex(tag => tag.name === tagName);
                            const tagId = this.keywordsLibrary[index].tagId;

                            return el.tags.includes(tagId)
                            }
                        )
                    );
                }

                return filteredLibrary;
            }
        },

        methods: {
            onRemove(lineSearch, item) {
                const index = this[lineSearch].indexOf(item);

                this[lineSearch].splice(index, 1);
            },

            onSelectedSprite(spriteName, event) {
                this.isSpriteSelected = event;
                this.nameSelectedSprite = spriteName;
            }
        }
    }

</script>

<style scoped>

    .search {
        position: fixed;
        padding: 10px 20px;
        width: 20vw;
    }

    .sprites {
        height: 80vh;
        overflow-y: auto;
        padding: 10px 10px 10px 20vw;
        display: flex;
        flex-flow: row wrap;
    }
</style>
