<template>
<loading v-if="loading"/>
<div v-else-if="board" class="ic-container">
    <div class="ic-paper round ic-z1 board-header" :style="lineStyle(board)">
        <div class="left">
            <h3 class="name">{{ board.name }}</h3>
            <div class="brief">{{ board.brief }}</div>
        </div>

        <div class="right">
            <router-link class="topic-new-btn" :to="{ name: 'forum_topic_new', params: {'board_id': board.id }}">发表主题</router-link>
        </div>
    </div>
    <div v-title v-if="$route.params.page && $route.params.page > 1">{{ board.name }} - 第{{$route.params.page}}页 - {{state.config.title}}</div>
    <div v-title v-else>{{ board.name }} - {{state.config.title}}</div>

    <div class="ic-xs ic-hidden xs-box">
        <router-link class="ic-btn primary" :to="{ name: 'forum_topic_new', params: {'board_id': board.id }}">发表主题</router-link>
    </div>

    <div class="board-page-box">
        <div class="topic-list" v-if="topics.items.length">
            <div class="board-item-box" :key="i.id" v-for="i in topics.items"  @mouseover="itemHover(i.id)" @mouseout="itemHover(null)">
                <router-link :to="{ name: 'forum_topic', params: {id: i.id} }" class="board-item" :class="{'top-post': i.sticky_weight}">
                    <div class="title" style="flex: 14 0 0%">
                        <h2>
                            <router-link :title="i.title" :to="{ name: 'forum_topic', params: {id: i.id} }">
                                <span>{{i.title}}</span>
                                <span v-if="i.state === state.misc.POST_STATE.CLOSE">[关闭]</span>
                            </router-link>
                            <span class="icons">
                                <i v-if="i.awesome == 1" class="icarus icon-diamond" title="优秀" style="color: #e57272" @click.prevent></i>
                                <i v-if="false" class="icarus icon-crown" title="精华" style="color: #e8a85d"></i>
                                <i v-if="isAdmin() && i.id === hoverId" class="icarus icon-sword-cross animated rotateIn" title="管理" style="color: #71c1ef; cursor: pointer" @click.prevent="setTopicManage(i)"></i>
                            </span>
                        </h2>
                        <p class="info">
                            <user-link :user="i.user_id" />  •  
                            <span> 发布于<ic-time :timestamp="i.time" /></span>  •  
                            <span>最后回复
                                <span v-if="i.s.last_comment_id">
                                    <user-link :user="i.s.last_comment_id.user_id" />
                                    <ic-time :timestamp="i.s.last_comment_id.time" />
                                </span>
                                <span v-else>从未</span>
                            </span>
                        </p>
                        <div class="icons">
                            <i v-if="i.sticky_weight" class="icarus icon-pin" title="置顶" />
                        </div>
                    </div>
                    <div class="detail ic-xs-hidden" style="flex: 4 0 0%">
                        <div class="count-block">
                            <div class="count">
                                <p class="num">{{i.s.click_count}}</p>
                                <p class="txt">点击</p>
                            </div>
                            <div class="count">
                                <p class="num">{{i.s.comment_count}}</p>
                                <p class="txt">回复</p>
                            </div>
                        </div>
                    </div>
                </router-link>
            </div>
            <paginator :page-info='topics' :route-name='"forum_board"' />
        </div>
        <div class="topic-list" v-else>还未有人发言 ...</div>

        <div class="board-info ic-xs-hidden">
            <div class="box" style="padding-left: 0px">
                <div class="board-note fade-transition" style="margin-top:5px">
                    <p><strong style="font-size: 16px">版块公告</strong></p>
                    <div v-if="board.desc" v-html="marked(board.desc || '')"></div>
                    <div v-else>版主很懒，什么也没有写</div>
                </div>
                <div class="others">
                    <template v-if="board.parent_id">
                        <p><strong>上级板块</strong></p>
                        <div class="ul-subboards">
                            <router-link class="item" :to="{ name: 'forum_board', params: {id: board.parent_id.id} }">
                                <div class="sign" :style="lineStyle(board.parent_id)"></div>
                                <span class="sub-board-item">{{board.parent_id.name}}</span>
                            </router-link>
                        </div>
                    </template>

                    <template v-if="siblingBoards">
                        <p><strong>同级板块</strong></p>
                        <div class="ul-subboards">
                            <router-link v-for="j in siblingBoards" :key="j.id" class="item" :to="{ name: 'forum_board', params: {id: j.id} }">
                                <div class="sign" :style="lineStyle(j)"></div>
                                <span class="sub-board-item">{{j.name}}</span>
                            </router-link>
                        </div>
                    </template>

                    <template v-if="subBoards">
                        <p><strong>子版块</strong></p>
                        <div class="ul-subboards">
                            <router-link v-for="j in subBoards" :key="j.id" class="item" :to="{ name: 'forum_board', params: {id: j.id} }">
                                <div class="sign" :style="lineStyle(j)"></div>
                                <span class="sub-board-item">{{j.name}}</span>
                            </router-link>
                        </div>
                    </template>
                </div>
            </div>
        </div>
    </div>
    <dialog-topic-manage />
</div>
<page-not-found v-else />
</template>

<style scoped>
.ul-subboards > .item {
    display: flex;
    align-items: center;
}

.ul-subboards > .item > .sign {
    width: 1em;
    height: 1em;
    background-color: #000;
    border-radius: 3px;
}

.ul-subboards {
    padding: 0;
    margin: 0;
    list-style: none;
}

.sub-board-item {
    margin-left: 5px;
}

.sub-board-item:not(:last-child)::after {
    content: ',';
}

.parent-link {
    font-weight: bolder;
    color: #fff;
}

.loading {
    display: flex;
    align-items: center;
    justify-content: center;
}

.board-header > .right {
    padding: 0 20px;
    display: flex;
    flex-direction: column;
    justify-content: center;
}

.xs-box {
    padding-top: 15px;
}

aside > .name {
    margin-top:0;
    font-size: 0.9em;
}

aside > .brief {
    font-size: 0.9em;
}

.board-header {
    background: #1f8dd6;
    padding: 1em 1em;
    border-radius: 3px;
    color: #fff;
    display: flex;
    justify-content: space-between;
}

.board-page-box {
    display: flex;
    margin-top: 10px;
    flex-direction: row;
}

.board-page-box > .topic-list {
    /* grow shrink basis*/
    flex: 18 0 0%;
}

.board-page-box > .board-info {
    flex: 6 0 0%;
}

.topic-new-btn {
    border: 1px solid #eee;
    color: #fff;
    padding: 5px 10px;
    border-radius: 4px;
    text-align: center;
}

.topic-new-btn:hover {
    border-color: #bbb;
}

.board-info > .box > .others {
    padding: 0 1em;
}

.board-note {
    padding: 0em 1em 1em 1em;
    font-size: 14px;
}

.board-note:hover {
    border-color: #bbb;
}
</style>

<script>
import api from '@/netapi.js'
import state from '@/state.js'
import {marked} from '@/md.js'
import '@/assets/css/_forum.scss'

export default {
    data () {
        return {
            state,
            hoverId: null,
            loading: true,
            board: null,
            subBoards: [],
            siblingBoards: [],
            topics: []
        }
    },
    methods: {
        marked,
        isAdmin: function () {
            return $.isAdmin()
        },
        setTopicManage: function (topic) {
            state.dialog.topicManageData = topic
            state.dialog.topicManage = true
        },
        lineStyle: function (board) {
            return $.lineStyle(board, 'background-color')
        },
        itemHover: function (id) {
            this.hoverId = id
        },
        test: function (id) {
            ;
        },
        fetchData: async function () {
            this.loading = true
            let params = this.$route.params
            let ret = await api.board.get({id: params.id, 'loadfk': {'parent_id': null}})

            if (ret.code) {
                // $.message_by_code(ret.code)
                this.loading = false
                return
            }

            let subBoards = await api.board.list({
                parent_id: params.id
            }, 1, null, state.getRole('user'))
            this.subBoards = subBoards.data.items

            let siblingBoards = await api.board.list({
                parent_id: ret.data.parent_id ? ret.data.parent_id.id : null,
                'id.ne': ret.data.id
            }, 1, null, state.getRole('user'))

            this.siblingBoards = siblingBoards.data.items

            let retList = await api.topic.list({
                board_id: params.id,
                order: 'sticky_weight.desc,weight.desc,update_time.desc',
                select: 'id, time, user_id, board_id, title, sticky_weight, state, awesome, update_time',
                loadfk: {'parent_id': null, 'user_id': null, 'id': {'as': 's', loadfk: {'last_comment_id': {'loadfk': {'user_id': null}}}}}
            }, params.page, null, state.getRole('user'))

            if (retList.code === api.retcode.SUCCESS) {
                this.board = ret.data
                this.topics = retList.data
            } else if (retList.code === api.retcode.NOT_FOUND) {
                this.board = ret.data
                this.topics = {'items': []}
            }
            this.loading = false
        }
    },
    created () {
        // 组件创建完后获取数据，
        // 此时 data 已经被 observed 了
        this.fetchData()
    },
    watch: {
        // 如果路由有变化，会再次执行该方法
        '$route': 'fetchData'
    },
    components: {
    }
}
</script>
