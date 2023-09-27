<template>
    <div class="lift">
        <div class="lift__container"
             v-for="(elevator, index) in arrayLifts"
             :key="index"
             :state="arrayLifts[index].stateLift"
        >
            <div class="lift__cabin"
                 :class="arrayLifts[index].stay ? 'flashing' : ''"
                 :style="{
                marginBottom: arrayLifts[index].marginBottom,
                transitionDuration: `${arrayLifts[index].distance}s`,
                transitionProperty: arrayLifts[index].marginBottom,
                height: `${height}px`
            }">
                <div class="lift__cabin_text">
                    <div>{{ arrayLifts[index].currentFloorText }}</div>
                    <div :class="arrowState(index)">↑</div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    import './lift.css'

    export default {
        // eslint-disable-next-line vue/multi-word-component-names
        name: 'Lift',
        props: {
            arrayLifts: Array,
            height: Number,
        },
        data() {
            return {};
        },
        methods: {
            arrowState(index) {
                switch (true) {
                    case (this.arrayLifts[index].stateLift === "down"):
                        return 'arrow-down'
                    case (this.arrayLifts[index].stateLift === "up"):
                        return 'arrow-up'
                    default:
                        return 'arrow'
                }
            },
        },
    };
</script>