<template>
    <div>
        <div>
            <Form ref="userForm" label-position="right">
                <FormItem :label="$t('general.name')" prop="name">
                    <div>
                        <span>{{ userForm.username }}</span>
                    </div>
                </FormItem>
                <FormItem :label="$t('general.real')" prop="name">
                    <div>
                        <span>{{ userForm.real_name }}</span>
                    </div>
                </FormItem>
                <FormItem :label="$t('general.department')">
                    <span>{{ userForm.department }}</span>
                </FormItem>
                <FormItem :label="$t('general.role')">
                    <span>{{ userForm.rule }}</span>
                </FormItem>
                <FormItem :label="$t('general.mail')">
                    <span>{{ userForm.email }}</span>
                </FormItem>
                <Button type="warning" size="small" @click="show_edit_password">{{$t('general.change_password')}}
                </Button>
                <Button type="primary" size="small" @click="openMailChange" class="margin-left-10">
                    {{$t('dash.edit_permissions')}}
                </Button>
                <Button type="success" size="small" @click="is_open = true" class="margin-left-10">
                    {{$t('general.show_permissions')}}
                </Button>
            </Form>
        </div>

        <Modal v-model="editEmailModal" :width="500" @on-ok="saveEmail">
            <h3 slot="header" style="color:#2D8CF0">{{$t('dash.edit_permissions')}}</h3>
            <Form :label-width="100" label-position="right">
                <FormItem :label="$t('general.mail')">
                    <Input v-model="editEmailForm.email"></Input>
                </FormItem>
                <FormItem :label="$t('general.real')">
                    <Input v-model="editEmailForm.real_name"></Input>
                </FormItem>
            </Form>
        </Modal>

        <edit_rule v-model="is_open"></edit_rule>
        <edit_password v-model="edit_password" :username="userForm.username"></edit_password>
    </div>
</template>

<script lang="ts">
    import edit_password from "@/components/modal/edit_password.vue";
    import edit_rule from "@/components/modal/edit_rule.vue";
    import att_mixins from "@/mixins/basic";
    import {Component, Mixins} from "vue-property-decorator";
    import module_verify from "@/store/modules/verify";
    import module_general from "@/store/modules/general";
    import module_user from "@/store/modules/user";

    @Component({components: {edit_password, edit_rule}})
    export default class personal extends Mixins(att_mixins) {
        editEmailModal = false;
        editEmailForm = {
            email: '',
            real_name: ''
        };
        userForm = {
            username: '',
            id: '',
            password: '',
            rule: '',
            department: '',
            real_name: '',
            email: ''
        };
        show_edit_password () {
            this.edit_password = !this.edit_password
            module_user.change_username(this.userForm.username)
        }
        openMailChange() {
            this.editEmailModal = true;
            this.editEmailForm = JSON.parse(JSON.stringify(this.userForm))
        }

        saveEmail() {
            this.$http.put(`${this.$config.url}/user/edit/mail`, {
                'mail': this.editEmailForm.email,
                'username': this.userForm.username,
                'real_name': this.editEmailForm.real_name
            })
                .then((res: { data: string; }) => {
                    this.$config.notice(res.data);
                    sessionStorage.setItem('real_name', this.editEmailForm.real_name)
                })
                .catch((error: any) => {
                    this.$config.err_notice(this, error)
                });
        }

        init() {
            this.$http.put(`${this.$config.url}/dash/profile`)
                .then((res: { data: { u: { username: string; id: string; password: string; rule: string; department: string; real_name: string; email: string; }; g: any; p: any; s: { Stmt: number; }; }; }) => {
                    this.userForm = res.data.u;
                    module_verify.fetch_user_permissions({
                        username: res.data.u.username,
                        list: res.data.g,
                        group: res.data.p
                    })
                    module_general.changed_stmt_status(res.data.s.Stmt === 0)
                })
        }

        mounted() {
            this.init()
        }
    }
</script>

<style lang="less">
    @import '../../styles/own-space.less';
    @import '../../styles/common.less';
</style>
