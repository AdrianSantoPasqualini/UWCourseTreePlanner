<template>
    <div class="homePanel">
        <SideBar />
        <Chart />
    </div>
</template>

<script>
import Chart from '../components/Chart';
import SideBar from '../components/SideBar';

export default {
    name: 'Home',
    components: {
        Chart: Chart,
        SideBar: SideBar,
    }
}
</script>

<style scoped>
.homePanel {
    height: 100%;
    display: flex;
}
</style>