<template>
    <div class="container">
        <div class="products-form">
            <h2>Products CRUD</h2>
            <input v-model="name" placeholder="name" />
            <input v-model="description" placeholder="description" />
            <input v-model="price" placeholder="price" />
            <button @click="handleSubmit" class="button">CREATE</button>
        </div>

        <div v-for="(product, i) in products" class="product-list">
            <div
                class="product"
                :class="{ 'product-alt': i % 2 === 0 }"
                @click="() => switchVisibility(product)"
            >
                <div class="product-wrapper">
                    <span>{{ product.name }}</span>
                    <img
                        @click="(evt) => deleteProduct(product, evt)"
                        v-bind:src="bin"
                        alt="bin"
                        class="bin"
                    />
                </div>
                <div v-if="product.visible" class="description">
                    <span>price: {{ product.price }}</span>
                    <span
                        >description:
                        {{
                            product.description
                                ? product.description
                                : 'no description'
                        }}</span
                    >
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios';
import bin from '../../../public/bin.png';

export default {
    data() {
        return {
            loading: false,
            error: null,
            products: [],
            name: '',
            description: '',
            price: null,
            bin,
        };
    },
    created() {
        this.fetchData();
    },
    methods: {
        async fetchData() {
            const res = await axios.post('/graphql', {
                query: `{
                    products {
                        id
                        name
                        description
                        price
                    }
                }`,
            });
            this.products = res.data.data.products;
        },

        async handleSubmit() {
            const desc =
                this.description === ''
                    ? ''
                    : `, description: "${this.description}"`;
            const res = await axios.post('/graphql', {
                query: `mutation {
                    createProduct(name: "${this.name}", price: ${this.price} ${desc}) {
                        id
                        name
                        description
                        price
                    }
                }`,
            });
            this.name = '';
            this.description = '';
            this.price = null;
            if (res.status >= 200 && res.status < 300) {
                this.products.push(res.data.data.createProduct);
            }
        },

        async deleteProduct(product, evt) {
            evt.stopPropagation();
            if (confirm(`Delete product ${product.name}?`)) {
                const res = await axios.post('/graphql', {
                    query: `mutation {
                    deleteProduct(id: ${product.id}) {
                        id
                    }
                }`,
                });
                if (res.status >= 200 && res.status < 300) {
                    this.products = this.products.filter(
                        (x) => x.id !== product.id
                    );
                }
            }
        },

        switchVisibility(product) {
            if (product.visible === true) product.visible = false;
            else product.visible = true;
        },
    },
};
</script>

<style scoped>
.product-list {
    display: flex;
    flex-direction: column;
    width: 100%;
    box-sizing: border-box;
}

.product-list:nth-child(odd) {
    background-color: #b6bdff;
}

.product {
    background-color: #e4c7ff;
    border-color: #3a3a3a;
    border-style: solid;
    border-width: 3px;
    border-top: none;
    position: relative;
    cursor: pointer;
    user-select: none;
    padding-left: 3px;
}

.product-alt {
    background-color: #b6bdff;
}

.products-form {
    display: flex;
    flex-direction: column;
    padding: 16px;
    background-color: #3a3a3a;
    width: 100%;
    box-sizing: border-box;
}

.product-wrapper {
    display: flex;
    justify-content: space-between;
    align-items: center;
    font-size: 1.2rem;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.description {
    display: flex;
    flex-direction: column;
    margin-top: 10px;
    gap: 4px;
}

.products-form > input {
    padding: 4px;
    font-size: 1.2rem;
    border-radius: 8px;
    margin-bottom: 8px;
}

.button {
    background-color: rgb(54, 255, 188);
    font-size: 1.1rem;
    border: none;
    border-radius: 8px;
    padding: 10px;
    cursor: pointer;
}

h2 {
    text-align: center;
    color: #fff;
}
.container {
    display: flex;
    align-items: center;
    justify-content: center;
    flex-direction: column;
    max-width: 900px;
    margin-left: auto;
    margin-right: auto;
}

.bin {
    width: 18px;
    height: 18px;
    padding: 6px;
}

.bin:hover {
    background-color: rgb(255, 64, 64);
}
</style>
