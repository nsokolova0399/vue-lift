<template>
    <div class="floor">
        <div v-for="floor in floors" class="floor__container" :key="floor">
            <div>{{ floor }}</div>
            <button :class="{ 'floor__container_button': activeButtons.includes(floor) }"
                    :style="{ 'background-color': activeButtons.includes(floor) ? 'green' : 'white' }"
                    @click="call(floor)">
                <i class="fa fa-dot-circle-o" aria-hidden="true"></i>
            </button>
        </div>
    </div>
</template>

<script>
    import './floor.css'
    export default {
        // eslint-disable-next-line vue/multi-word-component-names
        name: "Floor",
        props: {
            floors: Array,
            activeButtons: Array,
        },
        methods: {
            call(floor) {
                this.$emit("call", floor);
            },
        },
    }
</script>