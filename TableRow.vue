<template>

    <v-row>
        <v-col cols="1" :class="{'ml-3': !row.optional.isOptional}">
            <p v-if="row.optional.isOptional">
                {{ row.mainLevelsIndex + ' - ' + row.optionalLevelsIndex }}
            </p>

            <select v-if="!row.optional.isOptional"
                    @change="$emit('change-row-number', +$event.target.value)">
                <option v-for="number of mainLevelsList"
                        :key="number"
                        :selected="number === row.mainLevelsIndex">
                    {{number}}
                </option>
            </select>
        </v-col>

        <v-col cols="1">
            <v-btn x-small color="blue-grey lighten-3"
                   :disabled="row.optional.isOptional || row.mainLevelsIndex === 1"
                   @click="$emit('change-row-position', -1)">
                <v-icon>mdi-chevron-up</v-icon>
            </v-btn>
            <v-btn x-small color="blue-grey lighten-3"
                   :disabled="row.optional.isOptional || row.mainLevelsIndex === lastMainLevelIndex"
                   @click="$emit('change-row-position', 1)">
                <v-icon>mdi-chevron-down</v-icon>
            </v-btn>
        </v-col>

        <v-col cols="1">
            {{ row.levelId }}
        </v-col>

        <v-col>
            <input type="checkbox"
                   v-if="i !== 0"
                   :checked="row.optional.isOptional"
                   @change="$emit('toggle-optional', $event.target.checked)"
            />

            <select v-if="row.optional.isOptional"
                    @change="$emit('change-parent-id', $event.target.value)">
                <option v-for="id of listAvailableId"
                        :key="id"
                        :selected="id === row.optional.parentId">
                    {{ id }}
                </option>
            </select>
        </v-col>

        <v-col>
            <p v-for="p in row.title" :key="p">
                {{ p }}
            </p>
        </v-col>

        <v-col>
            <p v-for="p in row.created" :key="p">
                {{ p }}
            </p>
        </v-col>

        <v-col>
            <p v-for="p in row.edited" :key="p">
                {{ p }}
            </p>
        </v-col>

        <v-col>
            <p v-for="p in row.tested" :key="p">
                {{ p }}
            </p>
        </v-col>

        <v-col cols="1">
            {{ status[row.status] }}
        </v-col>

        <v-col cols="1">
            <p>
                {{ row.availability.isAvailability }}
            </p>
            <p v-if="row.availability.isAvailability">
                {{ row.availability.text }}
            </p>
        </v-col>

        <v-col cols="1">
            <v-btn small icon color="green" @click="editLevel(row.levelId, row.mainLevelsIndex)"
                   :disabled="!row.availability.isAvailability"
            >
                <v-icon>mdi-lead-pencil</v-icon>
            </v-btn>

            <v-btn small color="red" icon
                   :disabled="row.optional.isOptional || hasChild || !row.availability.isAvailability"
                   @click="deleteLevel(row.levelId, row.mainLevelsIndex)">
                <v-icon>mdi-delete</v-icon>
            </v-btn>

            <v-dialog v-model="JSONModal"
                      width="60vw"
                      height="60vh">
                <template v-slot:activator="{ on }">
                    <v-btn v-on="on"
                           small color="blue" icon
                           :disabled="!row.availability.isAvailability"
                           @click="onLoadJSON(row.levelId)">
                        <v-icon>mdi-json</v-icon>
                    </v-btn>
                </template>

                <json-editor :levelId="row.levelId"
                             :JSONText="JSONText"
                             @close-modal="JSONModal = false">
                </json-editor>

            </v-dialog>

        </v-col>

    </v-row>

</template>

<script>
    import status from '@/data/statusList.js';

    import JSONEditor from '@/components/JSONEditor.vue';

    import fb from 'firebase';

    export default {
        name: 'TableRow',
        components: {
            'json-editor': JSONEditor
        },
        props: {
            'row': Object,
            'mainLevelsList': Array,
            'listAvailableId': Array,
            'i': Number,
            'lastMainLevelIndex': Number,
            'hasChild': Boolean,
        },
        data() {
            return {
                status: status,

                JSONModal: false,
                JSONText: {value: '{}'}
            }
        },

        methods: {
            editLevel(levelId, mainLevelsIndex) {
                this.$router.push({name: 'Editor', query: {levelId}})
            },

            deleteLevel(levelId, mainLevelsIndex) {
                const answer = confirm('Вы действительно хотите удалить уровень?');

                if (!answer) return;

                let batch = fb.firestore().batch();

                /* удаляем текущий уровень*/
                const ref = fb.firestore().collection('Levels').doc(levelId);
                batch.delete(ref);

                /* удаляем из коллекции главных уровней текущий уровень*/
                const ref1 = fb.firestore().collection('MainLevelsOrder').doc(levelId);
                batch.delete(ref1);

                /* перезаписываем все последующие */
                fb.firestore().collection('MainLevelsOrder').where("mainLevelsIndex", ">", mainLevelsIndex)
                    .get()
                    .then(querySnapshot => {
                        let nextIndex = mainLevelsIndex;

                        querySnapshot.forEach(doc => {
                            let ref = fb.firestore().collection('MainLevelsOrder').doc(doc.id);
                            batch.update(ref, {
                                mainLevelsIndex: nextIndex
                            });

                            nextIndex += 1;
                        });

                        batch.commit()
                            .then(() => console.log('Main level has been deleted successfully'))
                            .catch(error => console.error("Error deleting main level: ", error));
                    })
                    .catch(error => console.log("Error getting documents: ", error));

            },

            onLoadJSON(levelId) {
                const storageRef = fb.storage().ref();

                const realLevelsRef = storageRef.child(`RealLevels`);
                const levelRef = realLevelsRef.child(levelId + '.json');

                levelRef.getDownloadURL()
                    .then((url) => {
                        fetch(url)
                            .then(res => res.json())
                            .then(data => this.JSONText.value = JSON.stringify(data, undefined, 4))
                            .catch(err => console.log(err))
                    })
                    .catch(err => {
                        switch (err.code) {
                            case 'storage/object-not-found': {

                                const data = this.JSONText.value;
                                const newFile = new Blob([data], {type: "application/json;charset=utf-8"});

                                levelRef.put(newFile)
                                    .then(() => {
                                        this.JSONText.value = '{}'
                                    })
                                    .catch(err => console.error('Error creating JSON: ', err));

                                break;
                            }
                        }
                    });
            }

        }
    }

</script>

<style scoped>

    .row {
        transition: all 0.5s;
        border: 1px solid #c0c0c0;
        flex-wrap: nowrap;
    }

    p {
        margin: 0;
    }

    input {
        margin-right: 10px;
    }

    select {
        border: 1px solid gray;
    }


</style>
