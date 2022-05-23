<template>
    <div class="bb-comment-header" v-if="data.attrs">
        <div class="bb-comment-header-top d-flex justify-content-between">
            <ul class="nav nav-tabs" role="tablist">
                <li class="nav-item">
                    <a @click.prevent="setActiveTab(commentsTab)"
                        :class="[this.activeTab === this.commentsTab ? 'active' : '']"
                        class="btn btn-primary"
                        style="padding: 10px;"
                        :href="'#' + commentsTab">
                        {{  data.attrs.count_all }} {{ __('Comments') }}
                    </a>
                </li>
                <li class="nav-item">
                    <a @click.prevent="setActiveTab(inviteTab)"
                        :class="[this.activeTab === this.inviteTab ? 'active' : '']"
                        class="btn btn-primary"
                        style="padding: 10px;"
                        @click="() => openInviteForm()"
                        :href="'#' + inviteTab">
                        Invite
                    </a>
                </li>
            </ul>
            
            <dropdown
                @click="() => !isLogged && openLoginForm()"
                icon="fas fa-comment"
                :selected="{name: isLogged ? userData.name : 'Login'}"
                :options="isLogged && [
                    {name: __('Logout'), onClick: logout}
                ]"
                :no-select-mode="true"
            />
        </div>
    </div>
</template>

<script>
import Http from '../../service/http';
import Ls from '../../service/Ls';
import Dropdown from "./Dropdown";
import StarRating from './StarRating';

export default {
    name: 'Header',
    components: {
        Dropdown,
        StarRating,
    },
    data: () => {
        return {
            isRecommended: false,
            countRecommend: 0,
        }
    },
    methods: {
        logout() {
            Http.post(this.logoutUrl).then(() => {
                Ls.remove('auth.token');
                this.getUser();
            });
        },
        async onRecommend() {
            const res = await Http.post(this.recommendUrl, { reference: this.reference });
            this.isRecommended = !res.data.data;
            this.countRecommend += this.isRecommended ? 1 : -1;
        }
    },
    mounted() {
        console.log("activeTab", this.activeTab);
    },
    props: {
        setActiveTab: {
            type: Function,
            required: true
        },
        activeTab: {
            type: String,
        }
    },
    computed: {
        isLogged() {
            return this.data.userData;
        },
        userData() {
            return this.data.userData ?? {}
        },
    },
    
    inject: ['reference', 'data', 'getUser', 'openLoginForm', 'openInviteForm', 'onChangeSort', 'logoutUrl', 'recommendUrl', 'commentsTab', 'inviteTab' ]
}
</script>
