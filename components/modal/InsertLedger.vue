<template>
    <div class="insert-ledger">
        <ul>
            <li
                v-for="(item, idx) in ledgerList"
                :key="idx"
            >
                <label>
                    사용처
                    <input
                        ref="place"
                        type="text"
                    >
                </label>
                <label>
                    :
                    <input
                        class="insert-ledger__price"
                        type="text"
                        @keydown.tab.prevent.stop="nextItem"
                        @keydown.enter="submitLedger"
                    >
                </label>
            </li>
        </ul>
        <button
            @click="submitLedger"
            type="button"
        >등록</button>
        <button
            @click="closeModal"
            type="button"
        >나가기</button>
    </div>
</template>

<script>
import { reactive, ref, onMounted } from '@vue/composition-api';

const submit = (emit) => {
    const submitLedger = () => {
        if (!confirm('가계부를 등록하시겠어요?')) return;
        emit('toggle', false);
    };
    return { submitLedger };
};

const insertLedger = (placeElem, root, emit) => {
    const defaultItem = { name: '', price: '0' };
    const ledgerList = reactive([defaultItem]);
    const nextItem = () => {
        ledgerList.push(defaultItem);
        root.$nextTick(() => {
            placeElem.value[placeElem.value.length - 1].focus();
        });
    };
    return { ledgerList, nextItem };
};

export default {
    name: 'InsertLedger',
    setup(_, { root, emit }) {
        const placeElem = ref(null);
        const { submitLedger } = submit(emit);
        const { ledgerList, nextItem } = insertLedger(placeElem, root);
        const closeModal = () => {
            if (!confirm('나가면 저장되지 않아요. 나가실건가요?')) return;
            emit('toggle', false);
        };
        onMounted(() => {
            placeElem.value[0].focus();
        });
        return { submitLedger, ledgerList, nextItem, place: placeElem, closeModal };
    },
};
</script>

<style lang="scss" scoped>
.insert-ledger {
    width: 100%;
    height: 100%;
    background-color: white;
}

.insert-ledger__price {
    text-align: right;
}
</style>