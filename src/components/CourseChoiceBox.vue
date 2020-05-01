<template>
    <div class="outer-choice-box">
        <div class="choice-title">
            <strong>
                {{choice.name}}
            </strong>
            {{ " - Select " + choice.amount + " of the following prerequisite courses:"}}
        </div>
        <div class="inner-choice-box">
            <button class="choice-item" 
                v-for="option in choice.options" :key="option.name"
                v-on:click="toggleSelected(option)"
                v-bind:class="{'selected':option.selected}">
                {{option.name}}    
            </button>
                
        </div>
    </div>
</template>

<script>
export default {
    name: "CourseChoiceBox",
    props: ["choice"],
    data() {
        return {
            totalSelected: 0,
        }
    },
    methods: {
        toggleSelected(option) {
            option.selected = !option.selected;
            if (option.selected) {
                this.totalSelected++;
            } else {
                this.totalSelected--;
            }
            if (this.totalSelected > this.choice.amount) {
                this.choice.options.some((currOption) => {
                    if (currOption.selected && currOption.name !== option.name) {
                        currOption.selected = false;
                        this.totalSelected--;
                    }
                    return this.totalSelected <= this.choice.amount;
                });
            }
        }
    }
}
</script>

<style scoped>
.outer-choice-box {
    border-radius: 0.5rem;
    border: 1px solid #e6e6e6;
    display: flex;
    flex-direction: column;
    padding-top: 5px;
    padding-bottom: 5px;
}
.inner-choice-box {
    display: flex;
    flex-wrap: wrap;
}
.choice-item {
    border-radius: 0.5rem;
    border: 1px solid #26C281;
    padding: 5px;
    margin: 5px;
    flex-grow: 1;
    font-size: 14px;
    background-color: white;
    transition-duration: 0.4s;
}
.choice-item:focus {
   outline: none;
   box-shadow: none;
}
.selected {
    background-color: #26C281;
    color: white;
}

.choice-title {
    margin-top: 5px;
    padding-left: 8px;
    text-align: left;
}

</style>