<template>
    <div class="outer-choice-box">
        <div class="choice-title">
            <strong>
                {{choice.parent + ": "}}
            </strong>
            {{"Select " + choice.options[0] + " of the following prerequisites"}}
        </div>
        <div class="inner-choice-box">
            <button class="choice-item" 
                v-for="option in choice.options.slice(1)" :key="option.name"
                v-on:click="toggleSelected(option)"
                v-bind:class="{'selected':selected.includes(option)}">
                {{option}}
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
            selected: []
        }
    },
    methods: {
        toggleSelected(option) {
            var change = {
                parent: this.choice.parent,
                unselected: 0,
                selected: 0,
            }
            if (this.selected.includes(option)) {
                this.selected = this.selected.filter(item => {
                    return item !== option;
                })
                change.unselected = option;
            } else {
                if (this.selected.length + 1 > this.choice.options[0]) {
                    change.unselected = this.selected.pop();
                }
                change.selected = option;
                this.selected.push(option);
            }
            this.$emit('choice-made', change)
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