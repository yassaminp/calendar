<template>
    <div
        class="d-flex align-items-center justify-content-center p-1 rounded"
        :class="{ 'border border-danger bg-light ': isSelected, 'text-danger': holiday }"
    >
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
        holiday: {
            type: Boolean,
            default: false,
        },
    },
};
</script>

<style></style>
