<template>
    <div class="app">
        <div class="app__container">
            <div :style="{ height: `${ height * numberFloor }px` }">
                <Lift
                        :arrayLifts="arrayLifts"
                        :height="height"
                />
            </div>
        </div>
        <div class="app__container_floor">
            <Floor
                    :floors="arrayFloors"
                    :active-buttons="buttonsActive"
                    @call="call"
            />
        </div>
    </div>
</template>

<script>
    import './app.css'
    import Lift from './components/Lift/Lift';
    import Floor from './components/Floor/Floor';

    // конфигурация приложения
    const config = {
        "numberFloor": 6,
        "numberLift": 2,
        "heightFloor": 100
    }

    export default {
        name: 'App',
        components: {
            Floor,
            Lift,
        },
        data() {
            return {
                numberFloor: config.numberFloor,
                numberLift: config.numberLift,
                height: config.heightFloor,

                queue: [],
                buttonsActive: [],
                arrayFloors: [],
                arrayLifts: [],
                floorBlocked: [],

                flashTime: 3000
            };
        },
        beforeMount() {
            for (let currentFloor = this.numberFloor; currentFloor >= 1; currentFloor--) {
                this.arrayFloors.push(currentFloor);
            }

            for (let i = 0; i < this.numberLift; i++) {
                const state = {
                    stay: false,
                    currentFloorText: 1,
                    distance: null,
                    prevFloor: 1,
                    marginBottom: 0,
                    currentFloor: 1,
                    stateLift: 'stopped',
                };
                this.arrayLifts.push(state);
            }
            // localStorage.removeItem('app')
            const savedState = JSON.parse(localStorage.getItem('app'));

            if (savedState) {
                for (let i = 0; i < this.numberLift; i++) {
                    this.arrayLifts[i].stay = savedState.stay[i];
                    this.arrayLifts[i].currentFloorText = savedState.currentFloorText[i];
                    this.arrayLifts[i].currentFloor = savedState.currentFloor[i];
                    this.arrayLifts[i].marginBottom = savedState.marginBottom[i];
                    this.arrayLifts[i].stateLift = savedState.stateLift[i];
                    this.arrayLifts[i].prevFloor = savedState.prevFloor[i];
                }
            }

            for (let i = 0; i < this.numberLift; i++) {
                this.floorBlocked.push(this.arrayLifts[i].currentFloor)
            }
            window.addEventListener('beforeunload', this.saveState);
        },

        beforeUnmount() {
            this.saveState();
            window.addEventListener('beforeunload', this.saveState);
        },
        methods: {
            call(requestedFloor) {
                if (this.floorBlocked.includes(requestedFloor)) {
                    return;
                }
                let nearestLiftIndex = -1;
                let nearestDistance = Number.MAX_VALUE;
                for (let i = 0; i < this.numberLift; i++) {
                    let distance = Math.abs(this.arrayLifts[i].currentFloor - requestedFloor);
                    if (this.arrayLifts[i].stateLift === 'stopped' && distance < nearestDistance) {
                        nearestLiftIndex = i;
                        nearestDistance = distance;
                    }
                }
                if (!this.queue.includes(requestedFloor)) {
                    this.queue.push(requestedFloor);
                }
                if (!this.buttonsActive.includes(requestedFloor)) {
                    this.buttonsActive.push(requestedFloor);
                }

                if (nearestLiftIndex >= 0 && this.queue.length > 0) {
                    this.run(nearestLiftIndex, requestedFloor);
                }
            },

            run(i, floor) {
                if (!this.queue.length) {
                    return;
                }
                this.floorBlocked.splice(this.floorBlocked.indexOf(this.arrayLifts[i].currentFloor), 1);
                this.floorBlocked.push(floor);

                this.arrayLifts[i].currentFloor = floor;
                this.queue.shift();
                this.arrayLifts[i].stateLift = parseInt(this.arrayLifts[i].marginBottom) > (this.arrayLifts[i].currentFloor - 1) * this.height ? 'down' : 'up';
                this.arrayLifts[i].marginBottom = `${(this.arrayLifts[i].currentFloor - 1) * this.height}px`;
                this.arrayLifts[i].distance = Math.abs(this.arrayLifts[i].currentFloor - this.arrayLifts[i].prevFloor);
                this.arrayLifts[i].currentFloorText = `${this.arrayLifts[i].currentFloor}`;

                setTimeout(() => {
                    this.arrayLifts[i].stay = true;
                    const index = this.buttonsActive.indexOf(this.arrayLifts[i].currentFloor);
                    if (index !== -1) {
                        this.buttonsActive.splice(index, 1);
                    }
                    this.arrayLifts[i].prevFloor = this.arrayLifts[i].currentFloor;
                    setTimeout(() => {
                        this.arrayLifts[i].stateLift = 'stopped';
                        this.arrayLifts[i].stay = false;
                        this.queue.length > 0 ? this.call(this.queue[0]) : null;
                    }, this.flashTime);
                }, this.arrayLifts[i].distance * 1000);
            },

            saveState() {
                const state = {
                    queue: this.queue,
                    buttonsActive: this.buttonsActive,
                    stay: [],
                    currentFloor: [],
                    prevFloor: [],
                    distance: [],
                    marginBottom: [],
                    currentFloorText: [],
                    stateLift: [],
                };
                for (let i = 0; i < this.numberLift; i++) {
                    state.currentFloorText.push(this.arrayLifts[i].currentFloor);
                    state.marginBottom.push(this.arrayLifts[i].marginBottom);
                    state.currentFloor.push(this.arrayLifts[i].currentFloor);
                    state.stateLift.push('stopped');
                    state.prevFloor.push(this.arrayLifts[i].prevFloor);
                }
                localStorage.setItem('app', JSON.stringify(state));
            },

        },
    }
</script>