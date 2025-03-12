<template>
    <div class="comments-tab">
        {{currentSpaceItem}}
        <ul class="comments-list">
            <li v-for="comment in comments" :key="comment.id">
                <strong>{{ comment.customer_firstname }} {{ comment.customer_lastname }}</strong>
                <span v-if="isCurrentUser(comment.customer_id)">(You)</span>
                <span v-if="comment.product_name">
          commented on
          <a :href="comment.product_url" target="_blank">{{ comment.product_name }}</a>
        </span>
                <small class="date">{{ comment.edited ? comment.formated_updated_at + ' (Edited)' : comment.formated_created_at }}</small>
                <div v-html="comment.comment"></div>

                <div v-if="comment.editable">
                    <button @click="editComment(comment)">Edit</button>
                    <button @click="deleteComment(comment.id)">Delete</button>
                </div>
            </li>
        </ul>

        <div class="new-comment-form">
            <textarea v-model="newComment" placeholder="Write your comment..."></textarea>
            <button @click="sendComment">Submit Comment</button>
        </div>
    </div>
</template>

<script>
import axios from 'axios';

export default {
    name: 'CommentsTab',
    data() {
        return {
            comments: [],
            newComment: '',
            projectId: null, // set accordingly
            formKey: null, // set accordingly
            currentSpace: { id: null }, // set accordingly
            currentSpaceItem: { id: null }, // set accordingly
        };
    },
    created() {
        this.loadComments();
    },
    methods: {
        buildUrl(path, params = {}) {
            const url = new URL(path, window.location.origin);
            Object.entries(params).forEach(([key, value]) => {
                url.searchParams.append(key, value);
            });
            return url.toString();
        },

        isCurrentUser(customer_id) {
            // Replace with your actual user-id checking logic
            return customer_id === this.currentCustomerId;
        },

        loadComments() {
            const url = this.buildUrl('/comments/presentation/load', {});

            axios.get(url)
                .then(({ data }) => {
                    if (Array.isArray(data.comments)) {
                        this.prepareCommentsData(data.comments);
                    } else {
                        console.warn('Unexpected response:', data);
                        this.comments = [];
                    }
                })
                .catch(error => {
                    console.error('Error loading comments:', error);
                });
        },

        prepareCommentsData(comments) {
            this.comments = comments.map(item => ({
                ...item,
                comment: item.comment.replace(/\n/g, '<br>')
            }));
        },

        sendComment() {
            if (!this.newComment.trim()) {
                alert('Please enter a comment.');
                return;
            }

            const url = this.buildUrl('/comments/presentation/save');
            const data = {
                project_id: this.projectId,
                comment: this.newComment,
                form_key: this.formKey,
                space_id: this.currentSpace.id,
                space_item_id: this.currentSpaceItem.id
            };

            axios.post(url, data)
                .then(({ data }) => {
                    if (data.success) {
                        this.newComment = '';
                        this.loadComments();
                    } else {
                        console.error('Failed to save comment:', data);
                    }
                })
                .catch(err => {
                    console.error('Error sending comment:', err);
                });
        },

        editComment(comment) {
            const updatedComment = prompt('Edit your comment:', comment.comment.replace(/<br>/g, '\n'));
            if (!updatedComment) return;

            const url = this.buildUrl('/comments/presentation/edit');
            const data = {
                project_id: this.projectId,
                comment_id: comment.id,
                comment: updatedComment,
                form_key: this.formKey,
                space_id: this.currentSpace.id,
                space_item_id: this.currentSpaceItem.id
            };

            axios.post(url, data)
                .then(({ data }) => {
                    if (data.success) {
                        this.loadComments();
                    } else {
                        console.error('Failed to edit comment:', data);
                    }
                })
                .catch(err => {
                    console.error('Error editing comment:', err);
                });
        },

        deleteComment(commentId) {
            if (!confirm('Are you sure you want to delete this comment?')) return;

            const url = this.buildUrl('/comments/presentation/delete');
            const data = {
                comment_id: commentId,
                project_id: this.projectId,
                form_key: this.formKey,
                space_id: this.currentSpace.id,
                space_item_id: this.currentSpaceItem.id
            };

            axios.post(url, data)
                .then(({ data }) => {
                    if (data.success) {
                        this.loadComments();
                    } else {
                        console.error('Failed to delete comment:', data);
                    }
                })
                .catch(err => {
                    console.error('Error deleting comment:', err);
                });
        }
    }
};
</script>
