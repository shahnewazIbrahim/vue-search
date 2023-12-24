<template>
    <div class="fixed inset-0 z-50">
        <div class="w-full h-full flex justify-center items-center">
            <div
                class="p-3 w-full mx-auto max-w-xs bg-white rounded border shadow z-50 overflow-scroll h-80"
            >
                <!-- @keyup="search" :value="$page.props.request[searchable] ?? ''" -->
                <div class="pb-2 sticky" v-if="multiple">
                    <small>Selected Item:</small>
                    <div class="grid grid-cols-3 gap-1">
                        <span
                            class="bg-gray-300 text-sm rounded py-1"
                            v-for="value in selected"
                            :key="value"
                        >
                            {{ data.elements[value]?.length > 8 ? `${data.elements[value].substring(0, 8)}...` : data.elements[value] }}
                            <sup
                                class="text-red-500 font-bold cursor-pointer"
                                @click="removeFromSelected(value)"
                                >X</sup
                            >
                        </span>
                    </div>
                </div>
                <input
                    placeholder="Search..."
                    v-model="seachvalue"
                    @keyup="search"
                    class="px-2 py-2 w-full rounded border border-gray-500 focus:outline-none focus:ring-0"
                />
                <div
                    class="px-2 py-1 border cursor-pointer"
                    :class="{
                        'bg-blue-500 text-white': selected.includes(index),
                    }"
                    v-for="(searchable, index) in dataValue"
                    :key="searchable.id ?? index"
                    @click="selectElement(searchable.id ?? index)"
                >
                    {{ searchable.name ?? searchable }}
                </div>
                <div
                    class="absolute right-2 top-0 p-1 cursor-pointer text-red-500 text-3xl z-40"
                    @click="close"
                >
                    &times;
                </div>
            </div>
            <div class="absolute inset-0 bg-gray-500 bg-opacity-50 z-40">
                <div class="w-full h-full" @click="close"></div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: "search",
    props: {
        data: {
            type: Object,
            default: {},
        },
        searchable: {
            type: String,
            default: "searchable",
        },
        multiple: {
            type: Boolean,
            default: false,
        },
    },
    data() {
        return {
            selected: this.$page.props.request.selected || [],
            seachvalue: "",
        };
    },
    computed: {
        dataValue() {
            let re = new RegExp(this.seachvalue.trim(), "gi");

            if (!(this.data.elements instanceof Array)) {
                let keys = Object.keys(this.data.elements);

                return keys
                    .filter((key) => this.data.elements[key].match(re))
                    .reduce(
                        (res, key) => (
                            (res[key] = this.data.elements[key]), res
                        ),
                        {}
                    );
            }

            return this.data.elements.filter((key) =>
                key.name ? key.name.match(re) : key.match(re)
            );
        },
    },
    methods: {
        // search(event) {
        // let params = {};
        // let url = this.data.paramId
        //     ? this.route(this.route().current(), this.data.paramId)
        //     : this.route(this.route().current());

        // if (event.target.value) {
        //     params = {
        //         ...this.$page.props.request,
        //         [this.searchable]: !event.target.value.includes("\\")
        //             ? event.target.value
        //             : "",
        //         // id: this.data.paramId || "",
        //     };
        // }
        // // url: https://inertiajs.com/manual-visits ,content : Browser history

        // this.$inertia.get(url, params, { preserveState: true });
        // },
        selectElement(element) {
            if (this.selected.includes(element)) {
                return this.selected.splice(this.selected.indexOf(element), 1);
            } else {
                this.selected.push(element);
            }
            console.log(this.selected);
            if (!this.multiple) {
                this.close();
            }
        },

        close() {
            delete this.$page.props.request[this.searchable];

            let url = this.data.paramId
                ? this.route(this.route().current(), this.data.paramId)
                : this.route(this.route().current());

            this.$inertia.get(
                url,
                { ...this.$page.props.request },
                { preserveState: true }
            );

            this.$emit("searchClose", {
                selected: this.selected,
            });
        },

        removeFromSelected(item) {
            let index = this.selected.indexOf(item);
            // console.log(this.selected);
            this.selected.splice(index, 1);
        },
    },
};
</script>
