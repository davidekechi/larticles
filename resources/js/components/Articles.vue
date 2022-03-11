<template>
    <div>
        <h1>Article</h1>
        <form @submit.prevent="addArticle()" class="mb-3">
            <div class="form-group">
                <label>Title</label>
                <input type="text" class="form-control" v-model="article.title" required>
            </div>

            <div class="form-group">
                <label>Body</label>
                <textarea type="text" class="form-control" v-model="article.body" required></textarea>
            </div>
            <button type="submit" class="btn btn-dark col-12 mt-3">Save</button>
        </form>
        <nav aria-label="Page navigation example">
            <ul class="pagination">
                <li :class="[{disabled: !pagination.prev_page_url}, 'page-item']">
                    <a class="page-link" href="#" @click="fetchArticles(pagination.prev_page_url)">Previous</a>
                </li>
                <li class="disabled page-item">
                    <a class="page-link text-dark" href="#">{{pagination.current_page}} of {{pagination.last_page}}</a>
                </li>
                <li :class="[{disabled: !pagination.next_page_url}, 'page-item']">
                    <a class="page-link" href="#" @click="fetchArticles(pagination.next_page_url)">Next</a>
                </li>
            </ul>
            </nav>
        <div class="card card-body mb-2" v-for="article in articles" v-bind:key="article.id">
            <h3>{{article.title}}</h3>
            <p>{{article.body}}</p>
            <hr/>
            <button class="btn btn-warning mb-3" @click="editArticle(article)">Edit</button>
            <button class="btn btn-danger" @click="deleteArticle(article.id)">Delete</button>
        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            articles: [],
            article: {
                id: '',
                title: '',
                body: ''
            },
            article_id: '',
            pagination: {},
            edit: false
        };
    },

    created() {
        this.fetchArticles();
    },

    methods: {
        fetchArticles(page_url){
            page_url = page_url || 'api/articles';
            let vm = this;
            fetch(page_url)
                .then(res => res.json())
                .then(res => {
                    this.articles = res.data;
                    vm.makePagination(res.meta, res.links);
                })
                .catch(err => console.log(err));
        },

        makePagination(meta, links){
            let pagination = {
                current_page: meta.current_page,
                last_page: meta.last_page,
                next_page_url: links.next,
                prev_page_url: links.prev
            }

            this.pagination = pagination;
        },

        addArticle(){
            if (this.edit === false) {
                fetch('api/article', {
                    method: 'POST',
                    headers: {
                        'Content-type': 'application/json',
                    },
                    body: JSON.stringify(this.article)
                })
                    .then(res => res.json())
                    .then(data => {
                        this.article.title = '';
                        this.article.body = '';
                        alert('Article Added!');
                        this.fetchArticles();
                    })
                    .catch(err => console.log(err));
            }else{
                fetch('api/article', {
                    method: 'PUT',
                    headers: {
                        'Content-type': 'application/json',
                    },
                    body: JSON.stringify(this.article)
                })
                    .then(res => res.json())
                    .then(data => {
                        this.article.title = '';
                        this.article.body = '';
                        alert('Article Updated!');
                        this.fetchArticles();
                    })
                    .catch(err => console.log(err));
            }
        },

        editArticle(article){
            this.edit = true;
            this.article.article_id = article.id;
            this.article.title = article.title;
            this.article.body = article.body;
        },

        deleteArticle(id){
            if (confirm('Are you sure')) {
                fetch(`api/article/${id}`, {
                    method: 'DELETE',
                })
                    .then(res => res.json())
                    .then(data => {
                        alert('Article Removed!');

                        this.fetchArticles();
                    })
                    .catch(err => console.log(err))
            }
        }
    }
}
</script>

