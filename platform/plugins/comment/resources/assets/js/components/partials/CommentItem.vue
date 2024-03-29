<template>
    <div class="bb-comment-item" v-show="data.userData && !deleted" :class="{'is-sending': comment.isSending}">
        <div class="d-flex">
            <avatar :user="comment.user"></avatar>

            <div class="bb-comment-content w-100">
                <div class="bb-comment-content-user">
                    <user-name :user="comment.user"></user-name>
                    <span class="badge badge-warning" v-if="comment.isAuthor">{{ __('Author') }}</span>
                    <span class="px-1">•</span>
                    <span class="time">{{ !comment.isSending ? comment.time : 'sending...' }}</span>
                </div>

                <div v-show="!showEdit">
                    <p v-html="linkify(comment.comment)"></p>

                    <div class="bb-comment-content-actions d-flex flex-wrap align-center">
                        <a class="reply" @click="replyIt" href="javascript:">{{ __('Reply') }}</a>
                        <span>•</span>
                        <a class="bb-like" :class="{'ok': comment.liked}" href="javascript:" @click="onLike">

                        </a>
                        <span>{{ comment.like_count }}</span>
                    </div>

                    <div class="mt-3 mb-4" v-if="showReply">
                        <comment-box :parent-id="comment.id" auto-focus=true />
                    </div>
                </div>
                <!-- Edit form -->
                <comment-box
                    auto-focus="true"
                    v-if="showEdit"
                    is-edit="true"
                    :default-value="comment.comment"
                    :on-success="onEditCommentSuccess"
                    :on-cancel="onCancel"
                    :parent-id="comment.parent_id"
                    :comment-id="comment.id"
                    :status = "comment.status"
                />


                <a class="load-more-replies" href="javascript:" @click="onLoadMore" v-if="comments.length && attrs.current_page < attrs.last_page">{{ __('Load more replies') }} ({{ attrs.total - comments.length }}) <i class="fas fa-chevron-down"></i></a>

                <div v-if="comments.length" class="mt-3">
                    <comment-item v-for="(item, index) in comments" :key="item.id" :comment="item" :on-delete-item="() => onDeletedItem(index)"  />
                </div>
            </div>
        </div>

        <div class="bb-comment-item-actions" v-if="data.userData && parseInt(comment.user_id) === parseInt(data.userData.id)">
            <a href="javascript:" class="ml-2" @click="onDelete">
                <i class="fas fa-trash-alt"></i>
            </a>
        </div>
    </div>
</template>

<script>
import Avatar from "./Avatar";
import CommentBox from "./CommentBox";
import UserName from "./UserName";
import StarRating from './StarRating';
import Http from '../../service/http';
import Ls from '../../service/Ls';
export default {
    name: 'CommentItem',
    data() {
        return {
            showReply: false,
            comments: [],
            attrs: {},
            showEdit: false,
            activePage: false,
            deleted: false,
        }
    },
    components: {
        Avatar,
        CommentBox,
        UserName,
        StarRating,
    },
    props: {
        comment: {
            type: Object,
            required: true,
        },
        onDeleteItem: {
            type: Function,
            required: true,
            default: () => null,
        }
    },
    mounted() {
        var self = this;
        window.Echo.channel('comment')
            .listen('.Botble\\Comment\\Events\\NewCommentEvent', (e) => {
                if(e.comment.parent_id == this.comment.id){
                    self.comments.unshift(e.comment);
                }
            });
            window.Echo.channel('like')
            .listen('.Botble\\Comment\\Events\\NewLikeEvent', (e) => {

                if( self.comment.id == e.commentId && !self.activePage){
                    self.comment.liked = !self.comment.liked;
                    self.comment.like_count += e.liked ? 1 : -1;
                }
            });
            window.Echo.channel('delete')
            .listen('.Botble\\Comment\\Events\\DeleteCommentEvent', (e) => {
                if(e.id == self.comment.id){
                    self.deleted = true;
                    self.attrs.count_all -= 1;
                }
            });

        const rep = this.comment.rep;
        if (rep && rep.data) {
            this.comments = rep.data.reverse();
            this.attrs = {
                total: rep.total,
                last_page: rep.last_page,
                current_page: rep.current_page,
                per_page: rep.per_page,
            }
        }
    },
    computed: {
        rated() {
            const rating = this.data.rating,
                comment = this.comment;

            if (
                rating &&
                comment &&
                rating.data[comment.user.id]
            ) {
                return rating.data[comment.user.id]
            }

            return 0;
        }
    },
    methods: {
        replyIt() {
            this.showReply = true;
        },
        onCancel() {
            this.showEdit = false;
            this.showReply = false;
        },
        onEditCommentSuccess(comment) {
            this.showEdit = false;
            this.comment = comment;
        },
        onDelete() {
            this.showConfirm(this.__('Confirm'), this.__('Are you sure that want to delete this comment?'), (ok) => {
                if (ok) {
                    this.setSoftLoading(true);
                    Http.delete(this.deleteUrl, {
                        params: {
                            id: this.comment.id,
                        }
                    }).then(res => {
                        const {data} = res;

                        this.setSoftLoading(false);

                        if (!data.error) {
                            this.updateCount(false);
                            this.onDeleteItem();
                        }
                    })
                }
            })
        },
        onDeletedItem(index) {
            this.comments.splice(index, 1)
        },
        onLoadMore() {
            this.apiLoadComments({
                up: this.comment.id,
                page: this.attrs.current_page + 1,
                limit: this.attrs.per_page,
            }, data => {
                this.comments = data.data.comments.reverse().concat(this.comments);
                this.attrs = data.data.attrs;
            })
        },
        onLike() {
            this.activePage = true;
            this.comment.liked = !this.comment.liked;
            this.comment.like_count += this.comment.liked ? 1 : -1;
            Http.post(this.likeUrl, {
                id: this.comment.id,
            }).then(() => {
               // this.activePage = true;
            });
        }
    },
    inject: ['data', 'deleteUrl', 'showConfirm', 'updateCount', 'apiLoadComments', 'setSoftLoading', 'likeUrl']
}
</script>
