<template>
    <div class="container column">
        <form class="card card-w30" @submit.prevent="addBlock">
            <div class="form-control">
                <label for="type">Тип блока</label>
                <select id="type" v-model="block.type">
                    <option value="title">Заголовок</option>
                    <option value="subtitle">Подзаголовок</option>
                    <option value="avatar">Аватар</option>
                    <option value="text">Текст</option>
                </select>
            </div>

            <div class="form-control">
                <label for="value">Значение</label>
                <textarea id="value" rows="3" v-model.trim="block.content"></textarea>
            </div>

            <button class="btn primary" type="submit" :disabled="!canAddBlock">Добавить</button>
        </form>

        <div class="card card-w70">
            <template v-if="blocks.length">
                <component :is="`block-${b.type}`" v-for="(b, $idx) in blocks" :key="$idx" :content="b.content"></component>
            </template>
            <h3 v-else>Добавьте первый блок, чтобы увидеть результат</h3>
        </div>
    </div>
    <div class="container">
        <p>
            <button class="btn primary" @click="loadComments">Загрузить комментарии</button>
        </p>
        <div class="card" v-if="comments.length">
            <h2>Комментарии</h2>
            <ul class="list">
                <li class="list-item" v-for="comment in comments" :key="comment.id">
                    <div>
                        <p>
                            <strong>{{ comment.email }}</strong>
                        </p>
                        <small v-html="comment.body"></small>
                    </div>
                </li>
            </ul>
        </div>
        <div v-if="loading" class="loader"></div>
    </div>
</template>

<script>
import AppLoader from './AppLoader';
import BlockTitle from './blocks/BlockTitle';
import BlockSubtitle from './blocks/BlockSubtitle';
import BlockAvatar from './blocks/BlockAvatar';
import BlockText from './blocks/BlockText';
import config from './config';
export default {
    components: {
        AppLoader,
        BlockTitle,
        BlockSubtitle,
        BlockAvatar,
        BlockText,
    },
    data() {
        return {
            comments: [],
            loading: false,
            block: {
                type: 'title',
                content: '',
            },
            blocks: ['title', 'avatar', 'subtitle', 'text'],
        };
    },
    created() {
        this.fetchBlocks();
    },
    methods: {
        async fetchBlocks() {
            try {
                const response = await fetch(config.url.blocks);
                const data = await response.json();
                this.blocks = Object.keys(data).map((idx) => {
                    return data[idx];
                });
            } catch (e) {
                // show alert or smth
                console.log(e);
            }
        },
        async loadComments() {
            this.loading = true;
            try {
                const response = await fetch(config.url.comments);
                this.comments = await response.json();
            } catch (e) {
                // show alert or smth
                console.log(e);
            }
            this.loading = false;
        },
        async addBlock() {
            try {
                await fetch(config.url.blocks, {
                    method: 'post',
                    headers: {
                        'Content-type': 'application/json',
                    },
                    body: JSON.stringify(this.block),
                });
                this.blocks.push({ ...this.block });
                this.block.content = '';
            } catch (e) {
                // show alert or smth
                console.log(e);
            }
        },
    },
    computed: {
        canAddBlock() {
            return this.block.content.length > 3;
        },
    },
};
</script>

<style>
.avatar {
    display: flex;
    justify-content: center;
}

.avatar img {
    width: 150px;
    height: auto;
    border-radius: 50%;
}
</style>
