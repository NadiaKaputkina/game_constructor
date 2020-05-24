<template>

    <v-card>
        <v-card-title
                class="headline grey lighten-2"
                primary-title>
            Новый спрайт
        </v-card-title>

        <v-text-field class="mx-4"
                      label="Название"
                      clearable
                      :rules="rules"
                      v-model="newSpriteName.name">
        </v-text-field>

        <v-combobox class="mx-4"
                    v-model="selectedTags"
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
                        @click:close="onRemove(item)">
                    {{ item }}
                </v-chip>
            </template>
        </v-combobox>

        <v-file-input class="mx-4"
                      @change="previewImage"
                      :rules="rules"
                      label="File input"
                      accept="image/png"
                      prepend-icon="mdi-camera"
        >
            <template v-slot:selection="{ text }">
                <v-chip small label>
                    {{ text }}
                </v-chip>
            </template>
        </v-file-input>

        <!--добавить-->
        <v-card-actions>
            <v-btn color="primary"
                   :disabled="!newSpriteName || !newFile"
                   @click="addNewSprite"
                   @click.close="$emit('close-modal', newSpriteName)">
                Добавить
            </v-btn>
        </v-card-actions>

    </v-card>


</template>

<script>

    import {ids} from '@/data/spritesTags'
    import fb from 'firebase';

    export default {
        name: "NewSprite",
        props: {
            keywordsLibrary: Array
        },
        data() {
            return {
                ids,

                newFile: '',

                newSpriteName: {name:''},
                newSpriteSrc: '',
                selectedTags: [],

                newKeywords: [],

                rules: [value => !!value]
            }
        },
   
        watch: {
            selectedTags: function (newArr, oldArr) {
                if (newArr.length > oldArr.length) {
                    const newKeywordName = newArr[newArr.length - 1];

                    const isKeywordExist = this.keywordsLibrary.some(el => el.name === newKeywordName);

                    if (!isKeywordExist) {
                        const newKeyword = {
                            tagId: '',
                            name: newKeywordName
                        };

                        fb.firestore().collection('SpritesKeywords').add(newKeyword)
                        .then(() => console.log('Keyword successfully added!'))
                        .catch(err => console.error('Error adding keyword: ', err));
                    }

                }
            }
        },

        computed: {
            filteredTags: function () {
                return this.keywordsLibrary.map(tag => tag.name);
            }
        },

        methods: {
            previewImage(event) {
                this.newFile = event;

                if (event !== undefined) {
                    this.newSpriteSrc = +(new Date()) + '_' + event.name;
                }
            },

            onRemove(item) {
                const index = this.selectedTags.indexOf(item);

                this.selectedTags.splice(index, 1);
            },

            addNewSprite() {
                const newSprite = {
                    spriteId: '',
                    name: this.newSpriteName.name,
                    id: '',
                    levelsId: [],
                    tags: [],
                    src: this.newSpriteSrc
                };

                newSprite.tags = this.selectedTags.map(tagName => {
                    const index = this.keywordsLibrary.findIndex(tag => tag.name === tagName);

                    return this.keywordsLibrary[index].tagId;
                });

                var storage = fb.storage();
                var storageRef = storage.ref();
                var listRef = storageRef.child('SpritesLibrary/' + this.newSpriteSrc);
                var file = this.newFile;

                listRef.put(file)
                .then( snapshot => {
  
                    snapshot.ref.getDownloadURL()
                        .then(downloadURL => {
                            newSprite.src = downloadURL;

                            fb.firestore().collection('SpritesLibrary').add(newSprite)
                            .then(() => {
                                this.newFile = [];
                                this.newSpriteName = {name:''};
                                this.selectedTags = [];

                            })
                            .catch(err => console.error('Error adding new sprite: ', err));
                        })
                });


            }
        }
    }
</script>

<style scoped>

</style>
