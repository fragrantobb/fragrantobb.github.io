<template>
    <div
        class="modal"
        v-if="active"
    >
        <div
            class="modal__background"
            @click="closeModal"
        ></div>
        <div class="modal__contents">
            <slot />
        </div>
    </div>
</template>

<script>
const modalActiveControll = (context) => {
    const closeModal = () => {
        context.emit('toggle', false);
    };
    return { closeModal };
};

export default {
    name: 'Modal',
    props: {
        active: { required: true, type: Boolean },
    },
    setup(props, context) {
        const { closeModal } = modalActiveControll(context);
        return { closeModal };
    },
}
</script>

<style lang="scss" scoped>
.modal {
    z-index: 1000;
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    transform: translate3d(0, 0, 0);
}

.modal__background {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.5);
}

.modal__contents {
    position: relative;
    width: 100%;
    height: 100%;
    background-color: white;
}
</style>