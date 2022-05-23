<template>
    <div class="bg-white br2 pa3 mt-3" style="box-shadow: 0 1px 1px #d9d9d9;">
        <ul class="nav" role="tablist" style="border-bottom: 1px solid #d9d9d9;">
            <li class="nav-item gray-text">
                <a @click.prevent="setActiveTab(registrationProfileTab)"
                   :class="[activeTab === registrationProfileTab ? 'show active b' : '']"
                   class="no-underline mr2 black-50 hover-black-70 pv1 ph2 db"
                   style="text-decoration: none; line-height: 32px;" data-bs-toggle="tab" :href="'#' + registrationProfileTab">{{
                    __('registration_profile') }}</a>
            </li>
        </ul>
        <div class="tab-content">
            <div :class="[activeTab === registrationProfileTab ? 'tab-pane fade show active' : 'tab-pane fade']"
                 :id="registrationProfileTab">
                 <div class="profile-item mt-3" v-for="(question, index) in questions" :key="index">
                    <p class="profile-item-header mb-0">{{ question.key }}</p>
                    <p class="profile-item-header mb-0" style="font-weight: 700;">{{ question.value }}</p>
                 </div>
            </div>
        </div>
        
    </div>
</template>

<script>
    export default {
        components: {
        },

        data() {
            return {
                activeTab: '',
                registrationProfileTab: 'registration-profile',
                isLoading: true,
                isLoadingMore: false,
                questions: [
                    {
                        'key': 'How soon do you plan to move?',
                        'value': 'Within 90 days'
                    },
                    {
                        'key': 'Are you working with real estate agent?',
                        'value': 'Not yet assigned'
                    },
                    {
                        'key': 'How would you describe your situation?',
                        'value': 'First time home buyer - I have no idea what I can afford'
                    },
                    {
                        'key': 'Please describe your reason for moving?',
                        'value': 'test'
                    },
                ]
            };
        },

        mounted() {
            this.prepareComponent();
        },

        props: ['defaultActiveTab'],

        methods: {
            prepareComponent() {
                this.setActiveTab(this.defaultActiveTab);
            },
            setActiveTab(tab) {

                if (this.activeTab === tab) {
                    return;
                }
    
                this.activeTab = tab;

            },
        }
    }
</script>

