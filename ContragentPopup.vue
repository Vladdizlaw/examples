<script setup>
import moderatorService from '@/services/api/moderatorService'
import { ElMessage } from 'element-plus'
import {
    Dialog,
    DialogPanel,
    DialogTitle,
    TransitionRoot,
    TransitionChild
} from '@headlessui/vue'
import { Search } from '@element-plus/icons-vue'
import { reactive, ref, watch } from 'vue'
const emit = defineEmits(['update:selected', 'update:open'])
const props = defineProps({ selected: { type: Object, default: {} }, open: { type: Boolean } })
const state = reactive({
    customers: [],
    user_id: undefined,
    loading: false,
    search: '',
    size: 50,
    scroll: 0,
    total: undefined,

})
const contragentsList = ref(null)
const isDebounse = { active: false, value: '' }
const handleOpenPopup = async () => {
    state.search = ''
    const params = {
        page: { size: state?.size }
    }
    await getAllCustomers(params)
    contragentsList.value.addEventListener('scroll', getScroll)
}
const getAllCustomers = async (params) => {
    state.loading = true
    try {
        const { data } = await moderatorService.contract.getContragents({ params })
        state.customers = data.contragents
        state.total = data.total
        state.loading = false
    } catch (e) {
        state.loading = false
        ElMessage({
            type: error, message: e.message
        })

    }
}

const closePopUp = () => {
    emit('update:open', false)
    contragentsList.value.removeEventListener('scroll', getScroll)
}
function onSelect(item) {
    emit('update:selected', item)
    closePopUp()

}
const getScroll = () => {
    state.scroll = contragentsList?.value?.scrollTop
}
watch(() => state.scroll, async (val) => {
    if ((val + contragentsList?.value?.clientHeight) >= (contragentsList?.value?.scrollHeight)) {
        if (state.size < state.total) {
            if (state.size + 50 > state.total) {
                state.size = state.total
            } else {
                state.size += 50
            }
            const params = {
                filter: {
                    q: state.search || undefined,
                },
                page: { size: state?.size }
            }
            await getAllCustomers(params)
        }
    }
})
watch(() => state.search, async (newVal) => {
    isDebounse.value = newVal
    setTimeout(async () => {
        const params = {
            filter: {
                q: isDebounse.value || undefined,
            },
            page: { size: state?.size }
        }
        await getAllCustomers(params)
    }, 500)
})
</script>
<template>
    <TransitionRoot appear :show="open" as="template">
        <Dialog as="div" :open="open" @close="closePopUp" :static="true">
            <TransitionChild as="template" enter="duration-300 ease-out" enter-from="opacity-0" enter-to="opacity-100"
                leave="duration-200 ease-in" leave-from="opacity-100" leave-to="opacity-0">
                <div class="fixed inset-0 bg-black bg-opacity-25 z-10" />
            </TransitionChild>
            <DialogPanel v-if="open" v-loading="state.loading"
                class="absolute z-10 top-0 bottom-0 right-0 left-0 m-auto rounded-md  drop-shadow-[5px_5px_10px_rgba(0,0,0,0.25)] flex flex-col p-4 min-w-[30rem] max-w-fit bg-white  min-h-[35rem]  max-h-[40vh] gap-2">
                <DialogTitle class="flex justify-start text-base font-bold">
                    <p class="flex gap-2 justify-center items-center">Контрагенты <span>({{ state.total || 0 }})</span> </p>
                </DialogTitle>
                <div class="flex flex-row justify-start items-center gap-2 text-base w-full">
                    <el-input placeholder="Поиск" size="medium" tabindex="-1" style="flex: 1" clearable
                        v-model="state.search">
                        <template #suffix>
                            <el-icon class="el-input__icon">
                                <search />
                            </el-icon>
                        </template>
                    </el-input>
                </div>
                <div ref="contragentsList"
                    class="h-full overflow-y-scroll w-full dialog flex flex-col justify-start border-[1px] border-solid border-gray-200 flex  rounded-md">
                    <div v-for="item of state.customers" :key="item.id" class="border-b cursor-pointer hover:bg-gray-100"
                        @click="onSelect(item)">
                        <h3 class="text-sm font-bold py-1 px-2">
                            {{ item.name }}
                            <p class="text-gray-600 text-xs font-normal pt-[2px]">
                                <span class="text-blue-500 "> ID:{{ item.id }}</span>, Инн:
                                {{
                                    item.inn
                                    ? item.inn
                                    : 'не указан'
                                }}
                            </p>
                        </h3>
                    </div>
                </div>
            </DialogPanel>
        </Dialog>
    </TransitionRoot>
</template>
<style lang="scss" scoped>
.dialog::-webkit-scrollbar {
    width: 6px;
    height: 6px;
    opacity: 0.5;

}

.dialog::-webkit-scrollbar-track {
    background-color: #F1F6FF;
    border-radius: 10px;
}

.dialog::-webkit-scrollbar-thumb {
    background: #D8D8D8;
    opacity: 0.6;
    border-radius: 10px;

}
</style>
