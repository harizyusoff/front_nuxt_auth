<template>
    <div class="l-content_maxWidth-sm mb-5 pb-5">
        <v-progress-linear
            :active="loading"
            :indeterminate="loading"
            absolute
            top
            color="orange white-4"
        />
        <br>
        <br>

        <br>
        <div class="v-stepper c-form_wrap">
            <v-container fluid>
                <div class="l-content_heading">
                    <h1>{{$t('signup.title')}}</h1>
                    <h4 class="slogan">
                        {{$t('signup.text')}}
                    </h4>
                </div>
                <vue-form-generator
                    ref="form"
                    :schema="schema"
                    :model="model"
                    class="c-form"
                    @model-updated="onInput"
                />

                <v-checkbox v-model="disabled" class="c-form_tnc">
                    <template v-slot:label>
                        <div>{{$t('common.agree')}}</div>
                    </template>
                </v-checkbox>
                <div class="text-center mb-5">
                    <button
                        type="submit"
                        block
                        x-large
                        class="c-btn c-btn_dark"
                        :disabled="!disabled"
                        @click="submitF()"
                    >
                        {{$t('common.submit')}}
                    </button
                    >
                </div>
            </v-container>
        </div>
    </div>
</template>

<script>
import '../assets/form.css';
import Vue from 'vue';
import VueFormGenerator from 'vue-form-generator';
import KurocoParser from '../plugins/parser.js';
import fieldUploadFile from '../components/vuetify_file_upload.vue';
import fieldUploadImage from '../components/vuetify_image_upload.vue';
import fieldVuetifyText from '../components/vuetify_text.vue';
import fieldVuetifyTextArea from '../components/vuetify_textarea.vue';
import fieldVuetifyDate from '../components/vuetify_date.vue';
import fieldVuetifyJson from '../components/vuetify_json.vue';
import fieldVuetifyPrefecture from '../components/vuetify_prefecture.vue';
import fieldVuetifyMultipleChoice from '../components/vuetify_multiple_choice.vue';
import fieldVuetifySingleChoice from '../components/vuetify_single_choice.vue';
import fieldVuetifySingleOption from '../components/vuetify_single_option.vue';
import fieldVuetifyPassword from '../components/vuetify_password.vue';

Vue.component('fieldUploadFile', fieldUploadFile);
Vue.component('fieldUploadImage', fieldUploadImage);
Vue.component('fieldVuetifyDate', fieldVuetifyDate);
Vue.component('fieldVuetifyText', fieldVuetifyText);
Vue.component('fieldVuetifyTextArea', fieldVuetifyTextArea);
Vue.component('fieldVuetifyJson', fieldVuetifyJson);
Vue.component('fieldVuetifyPrefecture', fieldVuetifyPrefecture);
Vue.component('fieldVuetifySingleOption', fieldVuetifySingleOption);
Vue.component('fieldVuetifySingleChoice', fieldVuetifySingleChoice);
Vue.component('fieldVuetifyMultipleChoice', fieldVuetifyMultipleChoice);
Vue.component('fieldVuetifyPassword', fieldVuetifyPassword);

Vue.use(VueFormGenerator);
Vue.use(KurocoParser);

export default {
    components: {
        'vue-form-generator': VueFormGenerator.component
    },
    methods: {
        onInput (value, fieldName) {
            this.$set(this.model, fieldName, value);
        },
        submitF () {
            const self = this;

            this.validForm = true;
            for (const key in self.$children[1].$children) {
                self.$children[1].$children[key].$children[0].$refs.myForm.validate();
                if (self.$children[1].$children[key].$children[0].formValid === false) {
                    this.validForm = false;
                }
            }

            if (this.validForm) {
                const sendModel = JSON.parse(JSON.stringify(self.model));
                this.loading = true;
                this.$auth.ctx.$axios
                    .post('/rcms-api/1/member/regist', sendModel)
                    .then(function (response) {
                        if (response.data.errors.length === 0) {
                            self.$store.dispatch('snackbar/setMessage', this.$i18n.t('signup.success'));
                            self.$store.dispatch('snackbar/snackOn');
                        }
                        self.loading = false;
                        self.$router.push('/');
                    })
                    .catch(function (error) {
                        self.$store.dispatch('snackbar/setError', error.response.data.errors?.[0].message);
                        self.$store.dispatch('snackbar/snackOn');
                        self.loading = false;
                    });
            } else {
                self.$store.dispatch('snackbar/setError', this.$i18n.t('verify.fille_property'));
                self.$store.dispatch('snackbar/snackOn');
                self.loading = false;
            }
        }
    },
    mounted() {},
    auth: false,
    data() {
        return {
            // form default values
            email: '',
            name1: '',
            name2: '',
            zip_code: '',
            tel: '',
            inquirySubmitUrl: '/rcms-api/1/inquiry/1',
            inquirySchemaUrl: '/rcms-api/1/inquiry/get/1',
            validForm: true,
            loading: false,
            disabled: false,
            model: {},
            schema: {
                fields: [
                    {
                        type: 'vuetifyText',
                        inputType: 'text',
                        min: 0,
                        max: 100,
                        label: this.$i18n.t('label.first_name'),
                        model: 'name1',
                        text: '',
                        required: true
                    },
                    {
                        type: 'vuetifyText',
                        inputType: 'text',
                        min: 0,
                        max: 100,
                        label: this.$i18n.t('label.last_name'),
                        text: '',
                        model: 'name2',
                        required: true
                    },
                    {
                        model: 'sex',
                        label: this.$i18n.t('label.gender'),
                        contents: [
                            {
                                key: 1,
                                value: 'Male',
                                default: false,
                                attribute: { group: '1' }
                            },
                            {
                                key: 2,
                                value: 'Female',
                                default: false,
                                attribute: { group: '1' }
                            },
                            {
                                key: 3,
                                value: 'Other',
                                default: false,
                                attribute: { group: '1' }
                            }
                        ],
                        required: true,
                        type: 'vuetifySingleChoice'
                    },
                    {
                        type: 'vuetifyDate',
                        inputType: 'picker',
                        label: this.$i18n.t('label.hire_date'),
                        model: 'hire_date',
                        required: true
                    },
                    {
                        type: 'vuetifyDate',
                        inputType: 'picker',
                        label: this.$i18n.t('label.birthday'),
                        model: 'birth',
                        required: true,
                        visible: (model, field, form) => model.choice === 'Check a boolean value'
                    },
                    {
                        type: 'vuetifyText',
                        inputType: 'text',
                        text: '',
                        min: 0,
                        max: 100,
                        label: this.$i18n.t('label.department'),
                        model: 'department',
                        required: false
                    },
                    {
                        type: 'vuetifyText',
                        inputType: 'text',
                        text: '',
                        min: 0,
                        max: 100,
                        label: this.$i18n.t('label.position'),
                        model: 'position',
                        required: false
                    },
                    {
                        type: 'vuetifyText',
                        inputType: 'text',
                        text: '',
                        min: 0,
                        max: 100,
                        label: this.$i18n.t('label.phone'),
                        model: 'tel',
                        texttype: 'tel',
                        required: true
                    },
                    {
                        type: 'vuetifyText',
                        inputType: 'text',
                        min: 0,
                        max: 100,
                        label: this.$i18n.t('label.email'),
                        model: 'email',
                        text: '',
                        texttype: 'email',
                        required: true
                    },
                    {
                        type: 'UploadFile',
                        label: this.$i18n.t('label.avatar'),
                        model: 'profileimage',
                        required: false,
                        visible: (model, field, form) => model.choice === 'Check a boolean value'
                    },
                    {
                        model: 'pull_down',
                        label: this.$i18n.t('label.office'),
                        contents: [
                            {
                                key: 1,
                                value: 'Tokyo',
                                default: false,
                                attribute: { group: '2' }
                            },
                            {
                                key: 2,
                                value: 'Osaka',
                                default: false,
                                attribute: { group: '2' }
                            },
                            {
                                key: 3,
                                value: 'Malaysia',
                                default: false,
                                attribute: { group: '2' }
                            }
                        ],
                        required: false,
                        type: 'VuetifySingleOption'
                    },
                    {
                        model: 'multiple_check',
                        label: this.$i18n.t('label.hobby'),
                        contents: [
                            {
                                key: 1,
                                value: 'Reading',
                                default: false,
                                attribute: { group: '2' }
                            },
                            {
                                key: 2,
                                value: 'Watching TV',
                                default: false,
                                attribute: { group: '2' }
                            },
                            {
                                key: 3,
                                value: 'Family Time',
                                default: false,
                                attribute: { group: '2' }
                            },
                            {
                                key: 4,
                                value: 'Going to Movies',
                                default: false,
                                attribute: { group: '2' }
                            },
                            {
                                key: 5,
                                value: 'Fishing',
                                default: false,
                                attribute: { group: '2' }
                            },
                            {
                                key: 6,
                                value: 'Computer',
                                default: false,
                                attribute: { group: '2' }
                            },
                            {
                                key: 7,
                                value: 'Gardening',
                                default: false,
                                attribute: { group: '2' }
                            },
                            {
                                key: 8,
                                value: 'Renting Movies',
                                default: false,
                                attribute: { group: '2' }
                            },
                            {
                                key: 9,
                                value: 'Walking',
                                default: false,
                                attribute: { group: '2' }
                            },
                            {
                                key: 10,
                                value: 'Exercise',
                                default: false,
                                attribute: { group: '2' }
                            }
                        ],
                        required: false,
                        type: 'vuetifyMultipleChoice'
                    },
                    {
                        model: 'notes',
                        type: 'vuetifyTextArea',
                        inputType: 'text',
                        label: this.$i18n.t('label.notes'),
                        placeholder: '',
                        text: '',
                        required: false,
                        counter: 400,
                        max: 400,
                        min: 0
                    }
                ]
            }
        };
    }
};
</script>
