<template>

    <v-card id="SpritesKeywords"
            class="overflow-auto px-3"
            height="300">
        <v-card-text>Список ключевых слов спрайтов:</v-card-text>

        <v-row class="px-3">
            <v-text-field label="Фильтр"
                          class="mr-4"
                          v-model="searchTag"
                          clearable>
            </v-text-field>

            <v-text-field label="Новое слово"
                          v-model="newKeyword"
                          clearable
                          :error-messages="isExist ? 'Keyword already exist!' : null"
                          :append-icon="'mdi-plus'"
                          @click:append="addNewKeyword"
                          @keyup.enter="addNewKeyword"
            ></v-text-field>

        </v-row>

        <v-row class="mx-0" v-for="(tag, i) of filteredTags" :key="i">
            <v-text-field :value="tag.name"
                          @input="changeValue($event, tag.name, i) "
                          dense>
            </v-text-field>

            <v-icon v-if="isEdit[i]"
                @click="updateTag(tag, i)">mdi-content-save</v-icon>
            <v-icon @click="deleteTag(tag)">mdi-delete</v-icon>
        </v-row>

    </v-card>

</template>

<script>
    import fb from 'firebase';

    export default {
        name: "KeywordsLibrary",
        props: {
            keywordsLibrary: Array
        },
        data() {
            return {
                searchTag: '',
                newKeyword: '',
                isExist: false,
                isEdit: [],
            }
        },

        computed: {
            filteredTags: function () {
                if (this.searchTag === null) return this.keywordsLibrary;

                return this.keywordsLibrary.filter(tag => tag.name.includes(this.searchTag))
            }
        },

        watch: {
            newKeyword: function (newWord, oldWord) {
                if (this.isExist && newWord !== oldWord) this.isExist = !this.isExist;
            }
        },

        methods: {
            changeValue(newValue, oldValue, i) {
                if (newValue !== oldValue) this.isEdit[i] = true;
            },

            updateTag(tag, i) {
                fb.firestore().collection('SpritesKeywords').doc(tag.tagId)
                .update({
                    name: tag.name
                })
                .then(() => {
                    this.isEdit[i] = false;
                    console.log("Keyword successfully update!")
                })
                .catch(error => console.error("Error updating keyword: ", error));
            },

            deleteTag(tag) {
                fb.firestore().collection('SpritesKeywords').doc(tag.tagId).delete()
                .then(() => console.log("Keyword successfully deleted!"))
                .catch(error => console.error("Error deleting keyword: ", error));

            },

            addNewKeyword() {
                if (this.newKeyword.length) {
                    const newKeyword = {
                        tagId: '',
                        name: this.newKeyword,
                    };

                    const isExist = this.keywordsLibrary.some(keyword => keyword.name === this.newKeyword);

                    if (!isExist) {
                        fb.firestore().collection('SpritesKeywords').add(newKeyword)
                        .then(() => console.log('Keyword successfully added!'))
                        .catch(err => console.error('Error adding keyword: ', err));

                        this.newKeyword = '';
                    } else {
                        this.isExist = true;
                    }
                }

            }
        }
    }
</script>

<style scoped>

</style>
