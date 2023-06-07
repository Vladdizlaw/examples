<script setup>
import HeaderList from '@/components/moderator/HeaderList.vue'
import { inject, reactive, computed, ref } from 'vue'
import Breadcrumbs from '@/components/ui/breadcrumbs'
import { ElMessage, ElLoading } from 'element-plus'
import moderatorService from '@/services/api/moderatorService.js'
import { useStore } from 'vuex'
import { useRouter } from 'vue-router'
import { vMaska } from "maska"

const router = useRouter()
const screenWidth = inject('screenWidth')
const crumbs = [
    { name: 'Пользователи', pathName: 'UsersList' },
    { name: 'Создание', pathName: '' }
]
const formAdditional = ref()

const state = reactive({
    form: {
        first_name: null,
        last_name: null,
        middle_name: null,
        phone: null,
        password: null,
        email: null,
        company_name: null,
        checkPassword: null,
        role: null
    },
    phoneMask: '+7 ### ###-##-##',
    errors: {},
    loading: false,
    timeouts: []

})
const roles = [{
    label: 'Заказчик', value: 'customer'
},
{
    label: 'Тайный покупатель', value: 'secretbuyer',
}]
const errors = {
    email: 'Введите корректный email',
    checkPassword: 'Пароль должен совпадать'
}
const rules = reactive({
    first_name: [
        { required: true, message: 'Введите имя, не менее 2 символов', trigger: 'blur', min: 2 }
    ],
    last_name: [
        { required: true, message: 'Введите фамилию, не менее 2 символов', trigger: 'blur', min: 2 }
    ],
    middle_name: [
        { required: false, message: 'Введите отчество, не менее 2 символов', trigger: 'blur', min: 2 }
    ],
    phone: [
        { required: true, message: 'Введите телефон ', trigger: 'blur', min: 11 }
    ],
    password: [
        { required: true, message: 'Введите пароль пользователя, не менее 8 символов', trigger: 'blur', min: 8 }
    ],
    checkPassword: [
        {
            required: true, message: "Повторите пароль", trigger: 'blur', min: 8
        }
    ],
    role: [
        { required: true, message: "Выберите роль" }
    ],
    email: [
        { required: true, message: 'Введите почту', trigger: 'blur' }
    ],
})


const setError = (field) => {
    state.errors[field] = errors[field]
    state.timeouts[field] = setTimeout(() => {
        delete state.errors?.[field]
    }, 2000)

}

const submitForm = async (form) => {
    state.errors = {}
    Object.keys(state.timeouts).forEach(t => {
        clearTimeout(state.timeouts[t])
    })
    await form.validate(async (valid) => {
        state.loading = true

        if (valid) {
            if (!state.form.email.split('@')[1]) {
                setError('email')
            }
            if (state.form.password !== state.form.checkPassword) {
                setError('checkPassword')
            }
            if (Object.values(state.errors).length) {
                state.loading = false
                return
            }
            try {
                await moderatorService.users.createUser(state.form).then(() => {
                    ElMessage.success('Пользователь создан')
                    state.loading = false
                    router.push({ name: 'UsersList' })
                })
            } catch (e) {
                state.loading = false
                ElMessage.error(e.message || e)
            }
        } else {
            ElMessage.error('Данные заполнены не корректно')
            state.loading = false
        }
    })
}
</script>

<template>
    <div class=" bg-[#E5E7EB] h-screen flex flex-col w-full  overflow-hidden ">
        <HeaderList :screenWidth="screenWidth">
            <template v-slot:title>
                <breadcrumbs :crumbs="crumbs" />
            </template>
            <template v-slot:button>
                <button v-loading="state.loading"
                    class="w-fit text-white font-[500] rounded-md px-4 py-1 bg-[#ed1115] hover:opacity-75"
                    @click="submitForm(formAdditional)">
                    Сохранить
                </button>
            </template>
        </HeaderList>

        <div class="h-[calc(100%-4rem)] w-full py-4 px-8 overflow-y-auto">
            <div class=" w-full flex flex-row gap-4 md:flex-nowrap flex-wrap justify-center">
                <el-form class="bg-white rounded-md shadow-md p-4   md:w-[68%] w-full" :rules="rules" ref="formAdditional"
                    label-position="top" label-width="120px" :model="state.form">

                    <div class="flex md:flex-row flex-col w-full gap-4 items-center justify-between">
                        <el-form-item required prop="last_name" class="flex flex-col md:w-[33%] w-full" label="Фамилия">
                            <el-input size="medium" filterable v-model="state.form.last_name" clearable class="w-full">
                            </el-input>
                        </el-form-item>

                        <el-form-item prop="first_name" class="md:w-[33%] w-full flex flex-col" label="Имя">
                            <el-input size="medium" filterable v-model="state.form.first_name" clearable class="w-full">
                            </el-input>
                        </el-form-item>

                        <el-form-item prop="middle_name flex flex-col" class="md:w-[33%] w-full" label="Отчество">
                            <el-input size="medium" filterable v-model="state.form.middle_name" clearable class="w-full">
                            </el-input>
                        </el-form-item>
                    </div>

                    <div class="w-full flex md:flex-row flex-col gap-4">
                        <el-form-item prop="email" class="w-full flex flex-row" label="Почта"
                            :error="state.errors['email']">

                            <el-input size="medium" filterable v-model="state.form.email" clearable class="w-full">
                            </el-input>
                        </el-form-item>
                        <el-form-item prop="phone" class="w-full flex flex-row" label="Телефон">
                            <el-input v-maska :data-maska="state.phoneMask" size="medium" filterable
                                v-model="state.form.phone" clearable class="w-full">
                            </el-input>
                        </el-form-item>
                    </div>
                    <div class="w-full flex md:flex-row flex-col gap-4">
                        <el-form-item prop="password" class="flex flex-col md:w-[59%]" label="Пароль">
                            <el-input size="medium" filterable v-model="state.form.password" clearable class="w-full">
                            </el-input>
                        </el-form-item>
                        <el-form-item prop="checkPassword" class="flex flex-col md:w-[59%]" label="Повторите пароль"
                            :error="state.errors['checkPassword']">
                            <el-input size="medium" filterable v-model="state.form.checkPassword" clearable class="w-full">
                            </el-input>
                        </el-form-item>
                    </div>
                    <div class="w-full flex md:flex-row flex-col gap-4">
                        <el-form-item prop="role" class="flex flex-col md:w-[49%]" label="Роль">
                            <el-select size="medium" filterable v-model="state.form.role" clearable class="w-full">
                                <el-option v-for="role, index in roles" :key="index" :label="role.label"
                                    :value="role.value" />
                            </el-select>
                        </el-form-item>
                        <el-form-item prop="company_name" class="flex flex-col md:w-[49%]" label="Название организации">
                            <el-input size="medium" filterable v-model="state.form.company_name" clearable class="w-full">
                            </el-input>
                        </el-form-item>
                    </div>

                </el-form>
            </div>
        </div>
    </div>
</template>
<style scoped lang="scss">
:deep.upload_form {
    height: fit-content;
    width: 100%;
    display: flex;
    gap: 0.5rem;

    .el-upload-list {
        width: 90%;
        display: flex;
        flex-direction: row;
        flex-wrap: wrap;
    }

    .el-upload-list__item {
        width: 100% !important;
        display: flex;
        align-items: center !important;

        .el-upload-list__item-info {
            width: 90% !important;
            word-break: break-all !important;
        }
    }


}
</style>
