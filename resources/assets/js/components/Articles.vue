<template>
    <div>
        <h2>Articles</h2>
        <form @submit.prevent="addArticle" class="mb-3">
            <div class="form-group">
                <input type="text" class="form-control" placeholder="Name" v-model="article.name">
                <textarea class="form-control" placeholder="Description" v-model="article.description"></textarea>
                <button type="submit" class="btn btn-light btn-block">Save</button>
            </div>
        </form>
        <nav aria-label="Page navigation example">
            <ul class="pagination">
                <li v-bind:class="[{disabled: pagination.prevPage<0}]" class="page-item"><a class="page-link" href="#"
                                                                                            @click="fetchArticles(pagination.prevPage)">Previous</a>
                </li>
                <li class="page-item disabled"><a class="page-link text-dark" href="#">Page {{ pagination.currentPage }} of {{ pagination.lastPage }}</a></li>
                <li v-bind:class="[{disabled: pagination.currentPage===pagination.lastPage}]" class="page-item"><a
                        class="page-link" href="#" @click="fetchArticles(pagination.nextPage)">Next</a></li>
            </ul>
        </nav>
        <div class="card card-body" v-for="article in articles" v-bind:key="article.code">
            <img :src="'https://d3el976p2k4mvu.cloudfront.net'+article.images[0].url" width="200px" height="150px">
            <!--<h3>{{ article.name }}</h3>-->
            <p><b>{{ article.manufacturerName }}: </b>{{ article.name }}</p>
            <p>price: <b>{{ article.price.supplementaryPriceLabel1 }}</b></p>
            <hr>
            <button @click="editArticle(article)" class="btn btn-primary">Edit</button>
            <button @click="deleteArticle(article.code)" class="btn btn-danger">Delete</button>
        </div>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                articles: [],
                article: {
                    code: '',
                    name: '',
                    description: ''
                },
                article_id: '',
                pagination: {},
                edit: false
            }
        },
        created() {
            this.fetchArticles();
        },
        methods: {
            fetchArticles(currentPage) {
                let vm = this;
                currentPage = currentPage || '0'
                fetch('https://www.maxi.rs/online/Pice%2C-kafa-i-caj/c/01/getSearchPageData?pageSize=12&pageNumber=' + currentPage + '&sort=promotion')
                    .then(res => res.json())
                    .then(res => {
                        this.articles = res.results;
                        vm.makePagination(res.pagination);
                    })
                    .catch(err => console.log(err));
            },
            makePagination(paginate) {
                let pagination = {
                    currentPage: paginate.currentPage,
                    lastPage: paginate.numberOfPages,
                    nextPage: paginate.currentPage + 1,
                    prevPage: paginate.currentPage - 1
                }

                this.pagination = pagination;
            },
            deleteArticle(id){
                if(confirm('Are you sure?')){
                    fetch(`api/article/${id}`,{
                        method: 'delete',
                    })
                        .then(res => res.json())
                        .then(data => {
                            alert('Article Removed');
                            this.fetchArticles();
                        })
                        .catch(err => console.log(err));
                }
            },
            addArticle(){
                if(this.edit === false){
                    //add
                    fetch('api/article',{
                        method: 'post',
                        description: JSON.stringify(this.article),
                        headers: {
                            'content-tye': 'application/json'
                        }
                    })
                        .then(res => res.json())
                        .then(data => {
                            this.article.name = '';
                            this.article.description = '';
                            alert('Article Added');
                            this.fetchArticles();
                        })
                        .catch(err => console.log(err));
                }else{
                    //update
                    fetch('api/article',{
                        method: 'put',
                        description: JSON.stringify(this.article),
                        headers: {
                            'content-tye': 'application/json'
                        }
                    })
                        .then(res => res.json())
                        .then(data => {
                            this.article.name = '';
                            this.article.description = '';
                            alert('Article Updated');
                            this.fetchArticles();
                        })
                        .catch(err => console.log(err));
                }
            },
            editArticle(article){
                this.edit = true;
                this.article.code = article.code;
                this.article.article_id = article.code;
                this.article.name = article.name;
                this.article.description = article.description;
            }
        }
    }
</script>