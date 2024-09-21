<template>
    <div class="d-flex align-items-center justify-content-center p-1 rounded" :class="{ isSelected: 'border border-danger bg-light ' }">
        {{ day }}
    </div>
</template>

<script>
export default {
    name: "VDate",
    props: {
        day: {
            type: Number,
            required: true,
        },
        isSelected: {
            type: Boolean,
            default: false,
        },
        icon: {
            type: String,
            default: "",
            required: false,
        },
    },
};
</script>

<style></style>
