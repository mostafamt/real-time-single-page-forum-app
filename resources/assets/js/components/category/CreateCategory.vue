<template>
    <v-container>

        <v-alert
        v-if="errors"
        :value="true"
        type="error"
        >
        This is a error alert.
        </v-alert>

        <v-form @submit.prevent="submit">
            <v-text-field
                label="Category Name"
                v-model="form.name"
                type="text"
                required
            ></v-text-field>

            <v-btn :disabled="disabled" v-if="editSlug" type="submit" color="pink">Update</v-btn>
            <v-btn :disabled="disabled" v-else type="submit" color="teal">Create</v-btn>
        </v-form>

        <v-card>
            <v-toolbar color="indigo" dark dense>
                <v-toolbar-title>Categories</v-toolbar-title>
            </v-toolbar>

            <v-list>
                <div v-for="(category, index) in categories" :key="category.id">
                    <v-list-tile>
                        <v-list-tile-action>
                            <v-btn icon small @click="edit(index)">
                                <v-icon color="orange">edit</v-icon>
                            </v-btn>
                        </v-list-tile-action>

                        <v-list-tile-content>
                            <v-list-tile-title>
                                {{category.name}}
                            </v-list-tile-title>
                        </v-list-tile-content>

                        <v-list-tile-action>
                            <v-btn icon small @click="destroy(category.slug, index)">
                                <v-icon color="red">delete</v-icon>
                            </v-btn>
                        </v-list-tile-action>

                    </v-list-tile>
                    <v-divider></v-divider>
                </div>
            </v-list>
        </v-card>

    </v-container>
</template>


<script>
export default {
    data(){
        return {
            form: {
                name: null,
            },
            categories: {},
            editSlug: null,
            errors: null,
        }
    },
    created(){
        if(!User.admin()){
            this.$router.push('/forum');
        }
        axios.get('/api/category')
        .then(res => this.categories = res.data.data);
    },
    computed: {
        disabled(){
            return !(this.form.name) ;
        }
    },
    methods: {
        submit(){
            this.editSlug ? this.update() : this.create() ;
        },
        destroy(slug, index){
            axios.delete(`/api/category/${slug}`)
            .then(res => this.categories.splice(index,1));
        },
        edit(index){
            this.form.name = this.categories[index].name ;
            this.editSlug = this.categories[index].slug ;
            this.categories.splice(index,1);
        },
        create(){
            axios.post('/api/category/', this.form)
            .then(res => {
                this.categories.unshift(res.data);
                this.form.name = null;
            })
            .catch(error => this.errors = error.response.data.error );
        },
        update(slug){
            axios.patch(`/api/category/${this.editSlug}`, this.form)
            .then(res => {
                this.categories.unshift(res.data);
                this.form.name = null;
                this.editSlug = null;
            })
        }
    },
}
</script>