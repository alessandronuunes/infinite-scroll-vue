<template>
    <div class="infi-scroll-comp-root">
        <div class="scroll-container" ref="scrollContainer" v-if="!initialLoad">
            <div class="list-container" ref="listContainer">
                <div
                    class="list-item"
                    v-for="(item, index) in list"
                    :key="index"
                >
                    {{ item.name }}
                </div>
            </div>
            <div class="sentinel" ref="sentinel"></div>

            <div v-if="canLoadMore" class="loadingMore">LOADING .....</div>
        </div>
        <div class="initialLoad" v-else>INITIAL LIST LOADING .....</div>
    </div>
</template>
<script setup>
    import { nextTick, ref , onBeforeUnmount, onMounted  } from 'vue'

    let listEndObserver
    const list =ref([])
    /**
     * if we extracted infinite-scroll to a generic component,
     * it would need just these props.loadingMore,canLoadMore and the list
     */
    const isLoadingMore = ref(false)
    const canLoadMore = ref(true) //list has to end at some point
    //extra stuff
    const initialLoad = ref(true)//initial load state ,if you want  to use a different BIGGER loading animation  etc
    const pageNumber = ref(0) //anLoadMore is set to false when last page is reached
    const pageCount = ref(10)
    const scrollContainer = ref(null)
    const sentinel = ref(null)

    function setUpInterSectionObserver() {
            let options = {
                root: scrollContainer.value,
                margin: "10px",
            };
            listEndObserver = new IntersectionObserver(
                handleIntersection,
                options
            );

            listEndObserver.observe(sentinel.value);
    }
    function handleIntersection([entry]) {
            if (entry.isIntersecting) {
                console.log("sentinel intersecting");
            }
            if ( entry.isIntersecting && canLoadMore.value && !isLoadingMore.value ) {
                loadMore();
            }
    }
    async function loadMore() {
            try {
                isLoadingMore.value = true;
                let items = await fetchItemsAPI( pageNumber.value, pageCount.value);
                console.log("loaded page " + pageNumber.value);
                pageNumber.value++;
                list.value.push(...items);
            } catch (error) {
                console.log("Reached end of page", error);
                canLoadMore.value = false;
            } finally {
                isLoadingMore.value = false;
            }
    }
    function fetchItemsAPI(pageNumber, pageCount) {
            return new Promise((res, rej) => {
                let data = [];
                for (let i = pageNumber * 10; i < pageNumber * 10 + 10; i++) {
                    data.push({
                        name: "Item " + i,
                    });
                }

                setTimeout(() => {
                    if (pageNumber < pageCount) {
                        res(data);
                    } else {
                        rej(new Error("No more items to load"));
                    }
                }, 1000);
            });
    }
    function initialize() {
        fetchItemsAPI(0, pageCount.value)
            .then((items) => {
                list.value.push(...items);
                pageNumber.value++;
                initialLoad.value = false;
                 //wait for initial list to render and then set up observer
                nextTick(() => setUpInterSectionObserver())
            });
    }
    onBeforeUnmount(() => {
        console.log('onBeforeUnmount')
        if (listEndObserver) {
            listEndObserver.disconnect();
        }
    })
    onMounted(() => {
        console.log('onMounted')
        initialize()
    })

</script>

<style lang="scss" scoped>
.scroll-container {
    height: 100%;
    overflow-y: scroll;
}
.sentinel {
    height: 0px;
}

.list-container {
    border: 1px solid darkgray;
}
.list-item {
    margin-bottom: 5px;
    border: 1px solid darkgoldenrod;
    background: lightgoldenrodyellow;
    color: black;
    padding: 15px;
    line-height: 25px;
    text-align: center;
    vertical-align: middle;
    font-size: 18px;
    border-radius: 5px;
}
.loadingMore {
    text-align: center;
}
</style>
