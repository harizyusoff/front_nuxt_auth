<template>
    <client-only>
        <div v-if="!auth.loggedIn" class="p-login_content elevation-7">
            <v-row>
                <v-col class="pa-0 col-sm-6 col-12">
                    <div class="p-login_intro">
                        <img src="~/assets/images/logo.png?width=150" class="p-login_logo">
                        <div class="p-login_intro-text">
                            <h1 class="heading">
                                <span v-html="$t('login.welcomeback')"></span>
                            </h1>
                            <p>{{$t('login.message')}}</p>
                            <p>
                                {{$t('login.demoaccount')}}<br>
                                <strong>{{$t('common.sitekey')}}：</strong> dev-nuxt-auth<br>
                                <strong>{{$t('common.id')}}：</strong>demo@kuroco-mail.app<br>
                                <strong>{{$t('common.password')}}：</strong>demo0512<br>
                            </p>
                        </div>
                    </div>
                </v-col>
                <v-col class="pa-0 col-sm-6 col-12">
                    <v-card class="p-login_form" outlined>
                        <form @submit.prevent="login">
                            <div class="login-screen lgn-left">
                                <v-card-title>
                                    <h2 align="center" class="pb-4 c-text_blue">
                                        {{ $t('common.sign_in') }}
                                    </h2>
                                </v-card-title>
                                <v-card-text class="inner">
                                    <form @submit.prevent="login">
                                        <v-text-field
                                            v-model="sitekey"
                                            :label="$t('login.site_key')"
                                            type="text"
                                            outlined
                                        />
                                        <v-text-field
                                            v-model="form.email"
                                            :label="$t('login.id_or_email')"
                                            type="email"
                                            outlined
                                        />
                                        <v-text-field
                                            v-model="form.password"
                                            :label="$t('common.password')"
                                            :type="show_pwd1 ? 'text' : 'password'"
                                            :append-icon="show_pwd1 ? 'mdi-eye' : 'mdi-eye-off'"
                                            outlined
                                            @click:append="show_pwd1 = !show_pwd1"
                                        />
                                        <p>
                                            <NuxtLink :to="localePath('/reminder')">
                                                {{ $t('login.forget_password') }}
                                            </NuxtLink>
                                        </p>
                                        <div class="text-center">
                                            <button
                                                type="submit"
                                                block
                                                x-large
                                                :loading="loading"
                                                class="c-btn_dark c-btn submit-btn"
                                            >
                                                {{ $t('common.sign_in') }}
                                            </button>
                                            <span v-html="$t('login.note')"></span>
                                        </div>
                                    </form>
                                </v-card-text>
                            </div>
                        </form>
                    </v-card>
                </v-col>
            </v-row>
        </div>

        <div v-else class="mypage">
            <v-carousel class="p-dashboard_banner">
                <v-carousel-item
                    v-for="(item, i) in items"
                    :key="i"
                    :src="item.src"
                    reverse-transition="fade-transition"
                    transition="fade-transition"
                />
            </v-carousel>

            <h1 class="text-left mt-5 pt-4">
                {{ $t('top.latest_articles') }}
            </h1>
            <v-topics :topics="topics" />

            <div class="text-center py-5 mt-4">
                <a
                    type="submit"
                    block
                    x-large
                    color="success"
                    class="c-btn c-btn_main c-btn_md c-btn_icon"
                    @click="back()"
                >
                    {{ $t('top.more_articles') }}
                    <v-icon
                        dark
                        right
                        class="icon"
                    >
                        mdi-arrow-right-drop-circle
                    </v-icon>

                </a>
            </div>

            <h1 class="text-left mt-5 pt-4">
                {{ $t('top.starred') }}
            </h1>
            <v-favourite :topics="favourite" />
            <div class="text-center py-5">
                <a
                    type="submit"
                    block
                    x-large
                    color="success"
                    class="c-btn c-btn_main c-btn_md c-btn_icon"
                    @click="linkFav()"
                >
                    {{ $t('top.more_starred') }}
                    <v-icon
                        dark
                        right
                        class="icon"
                    >
                        mdi-arrow-right-drop-circle
                    </v-icon>

                </a>
            </div>
        </div>
    </client-only>
</template>

<script>
import topicList from '../components/topics';
import topicGrid from '../components/topics_grid';

export default {
    components: {
        'v-topics': topicGrid,
        'v-favourite': topicList
    },
    middleware: 'auth',
    auth: false,
    created() {
        const myBody = document.getElementsByTagName('body')[0];
        if (this.$auth.loggedIn) {
            myBody.classList.add('p-dashboard');
            myBody.classList.remove('p-login');
        } else {
            myBody.classList.remove('p-dashboard');
            myBody.classList.add('p-login');
        }
        if (this.$route.name === 'index') {
            myBody.classList.add('p-home');
        }
    },
    data: () => ({
        sitekey: '',
        loading: false,
        show_pwd1: false,
        show_pwd2: false,
        form: {
            email: '',
            password: ''
        },
        items: [],
        thumbnail: [],
        topics: [],
        favourite: [],
        maxFavPerPage: 5 // This is fav list, for latest topic, go to below updateTopics() section, search for cnt=6
    }),
    computed: {
        user() {
            return this.$auth.user;
        },
        auth() {
            return this.$store.$auth;
        }
    },
    mounted() {
        this.topics = [];
        this.favourite = [];
        if (this.$auth.loggedIn) {
            this.updateTopics();
            this.updateFavourite();
        }
        const sitekey = localStorage.getItem('sitekey');
        if (sitekey != '') {
            this.sitekey = sitekey;
        } else {
            this.sitekey = 'dev-nuxt-auth';
        }
    },
    methods: {
        back() {
            this.$router.push(this.localePath('/topics_list/'));;
        },
        linkFav() {
            this.$router.push(this.localePath('/favourite/'));;
        },
        updateDesign() {
            const myBody = document.getElementsByTagName('body')[0];
            myBody.classList.add('p-dashboard');
            myBody.classList.remove('p-login');
        },
        updateTopics() {
            const url =
        '/rcms-api/1/topics?cnt=6';
            const self = this;
            this.$store.$auth.ctx.$axios
                .get(url)
                .then(function (response) {
                    const topics = [];
                    self.items = [];
                    for (const key in response.data.list) {
                        const item = response.data.list[key];
                        let fileurl = '';
                        let linkurl = '';
                        let thumbnail = '';
                        if (
                            item.hasOwnProperty('ext_col_02') &&
              item.ext_col_02.hasOwnProperty('url')
                        ) {
                            fileurl = item.ext_col_02.url;
                        }
                        if (
                            item.hasOwnProperty('ext_col_03') &&
              item.ext_col_03.hasOwnProperty('url')
                        ) {
                            linkurl = item.ext_col_03.url;
                        }
                        if (
                            item.hasOwnProperty('ext_col_08')
                        ) {
                            thumbnail = item.ext_col_08.url;
                        }
                        if (
                            item.hasOwnProperty('ext_col_08') &&
              item.ext_col_08.hasOwnProperty('url')
                        ) {
                            self.items.push({ src: item.ext_col_08.url + '?height=500px' });
                        }
                        topics.push({
                            date: item.ymd.substring(0, 10).replaceAll('-', '/'),
                            label: item.contents_type_nm,
                            link: item.subject,
                            id: item.topics_id,
                            icon: item.ext_col_01.key,
                            fileurl,
                            linkurl,
                            thumbnail,
                            edit: false
                        });
                    }
                    self.topics = topics;
                })
                .catch(function (error) {
                    self.$store.dispatch('snackbar/setError', error.response.data.errors?.[0].message);
                    self.$store.dispatch('snackbar/snackOn');
                });
        },
        updateFavourite() {
            const self = this;
            const favoritesUrl =
          '/rcms-api/1/favorites?member_id=' +
          this.$auth.user.member_id +
          '&module_type=topics';
            this.$store.$auth.ctx.$axios
                .get(favoritesUrl)
                .then(function (response) {
                    const topicIds = [];
                    for (const key in response.data.list) {
                        const item = response.data.list[key];
                        if (item.hasOwnProperty('module_id')) {
                            topicIds.push(item.module_id);
                        }
                    }
                    let url =
            '/rcms-api/1/topics?pageID=' +
            self.page +
            '&cnt=' +
            self.maxFavPerPage;

                    if (topicIds.length > 0) {
                        for (let i = 0; i < topicIds.length; ++i) {
                            url += '&id[]=' + topicIds[i];
                        }

                        self.$store.$auth.ctx.$axios
                            .get(url)
                            .then(function (response) {
                                const topics = [];
                                self.totalCnt = response.data.pageInfo.totalCnt;
                                for (const key in response.data.list) {
                                    const item = response.data.list[key];
                                    topics.push({
                                        date: item.inst_ymdhi
                                            .substring(0, 10)
                                            .replaceAll('-', '/'),
                                        label: item.contents_type_nm,
                                        link: item.subject,
                                        icon: '',
                                        id: item.topics_id,
                                        edit: false
                                    });
                                }
                                self.favourite = topics;
                            })
                            .catch(function (error) {
                                self.$store.dispatch('snackbar/setError', error.response.data.errors?.[0].message);
                                self.$store.dispatch('snackbar/snackOn');
                            });
                    }
                })
                .catch(function (error) {
                    self.$store.dispatch('snackbar/setError', error.response.data.errors?.[0].message);
                    self.$store.dispatch('snackbar/snackOn');
                });
        },
        async login() {
            this.loading = true;
            localStorage.setItem('sitekey', this.sitekey); // save sitekey
            if (this.sitekey === 'dev-nuxt-auth') {
                this.$store.$auth.ctx.$axios.defaults.baseURL = 'https://dev-nuxt-auth.a.kuroco.app';
            } else {
                this.$store.$auth.ctx.$axios.defaults.baseURL = 'https://' + this.sitekey + '.g.kuroco.app';
            }
            await this.$auth
                .loginWith('local', { data: this.form })
                .then(() => {
                    this.updateTopics();
                    this.updateDesign();
                    this.$router.push(this.localePath('/'));
                    this.$store.dispatch('snackbar/setMessage', this.$i18n.t('slackbar.logged_in'));
                    this.$store.dispatch('snackbar/snackOn');
                    this.loading = false;
                })
                .catch(() => {
                    this.$store.dispatch('snackbar/setError', this.$i18n.t('slackbar.login_fail'));
                    this.$store.dispatch('snackbar/snackOn');
                    this.loading = false;
                });
        }
    }
};
</script>

