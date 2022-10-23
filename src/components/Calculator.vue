<style>
/* Font */
div {
    font-family: Arial, Helvetica, sans-serif;
}

input {
    font-family: Arial, Helvetica, sans-serif;
}

button {
    font-family: Arial, Helvetica, sans-serif;
}

/* Phones */
@media (max-width: 512px) {
    .flex-start-center {
        display: block;
    }
}
</style>
<template>
    <div style="width:fit-content">
        <div>
            <h1 class="mb-half">Welcome to the Equitable Receipt Calculator!</h1>
            <div v-show="showExplanation">
                More simply put, a calculator to divide your receipt between friends/lovers/co-workers.<br>
                It equitably splits the bill, including tip and tax included, <br>
                based on how much each person paid for.<br>
                No data is stored, as all calculations occurs in your browser.<br>
                Check out the source code <a href="https://github.com/danjchoi/shared-receipt-calc">here</a>!
            </div>
            <div class="flex-center-end mr-1-5">
                <button v-show="!showExplanation" @click="showExplanation=true" class="button">What is this?</button>
                <div v-show="showExplanation" @click="showExplanation=false" class=button>Okay, got it!</div>
            </div>
        </div>
        <br>
        <div>
            <div class="mb-half flex-center-between">
                <div class="mr-half text-bold">Who are you paying?</div>
                <div class="mr-1-5">
                    <input v-model="recipient" type="text" placeholder="Name" autofocus>
                </div>
            </div>
            <div class="mb-half">
                <div class="flex-center-between">
                    <div class="mr-half mb-half text-bold">
                        Paying vagabonds:
                    </div>
                    <button v-show="vagabonds.length == 0" @click="addVagabond" class="button mr-1-5">
                        Add a vagabond
                    </button>
                </div>
                <div>
                    <div class="ml-1" v-for="vagabond, i in vagabonds">
                        <div class="flex-center-between mb-half">
                            <div class="mr-half">
                                <label :for="'name' + i">Vagabond {{i + 1}}:</label>
                            </div>
                            <div class="flex-center-end">
                                <div>
                                    <input :id="'name' + i" v-model="vagabond.name" type="text" placeholder="Name">
                                </div>
                                <div class="ml-half">
                                    <img @click="removeVagabond(i)" src="trash.png" class="clear-button">
                                </div>
                            </div>
                        </div>
                        <div>
                            <div class="flex-center-between mb-half" v-for="item, j in vagabond.items">
                                <div class="ml-1 flex-center-center">
                                    <div>
                                        <label :for="'item' + j" class="mr-half">Item:</label>
                                    </div>
                                    <div>
                                        <input v-model="item.name" type="text" placeholder="Name">
                                    </div>
                                </div>
                                <div class="flex-center-end ml-half">
                                    <div>
                                        $&nbsp;<input v-model="item.price" type="number" placeholder="Price" min="0"
                                            step="0.01">
                                    </div>
                                    <div class="ml-half">
                                        <img @click="removeItem(i, j)" src="trash.png" class="clear-button">
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="flex-center-end">
                            <button @click="addItem(vagabond)" class="button mr-1-5">
                                Add item
                            </button>
                        </div>
                    </div>
                </div>
                <div v-show="vagabonds.length != 0" class="flex-center-end">
                    <button @click="addVagabond" class="button mr-1-5">
                        Add vagabond
                    </button>
                </div>
            </div>
            <div class="mb-half">
                <div class="flex-center-between">
                    <div class="mb-half text-bold">Shared Items:</div>
                    <div v-show="sharedItems.length == 0" class="flex-center-end mr-1-5">
                        <button @click="addSharedItem" class="button">
                            Add shared item
                        </button>
                    </div>
                </div>
                <div v-for="item, i in sharedItems" class="flex-center-between mb-half">
                    <div class="ml-1 mr-half flex-center-center">
                        <div>
                            <label :for="'shared_item' + i" class="mr-half">Item:</label>
                        </div>
                        <div class="mr-half">
                            <input v-model="item.name" type="text" placeholder="Name">
                        </div>
                    </div>
                    <div class="flex-center-end" style="width:fit-content">
                        <div class="flex-center-end">
                            <div>
                                $&nbsp;
                            </div>
                            <div>
                                <input v-model="item.price" type="number" placeholder="Price" min="0" step="0.01">
                            </div>
                        </div>
                        <div class="ml-half">
                            <img @click="removeSharedItem(i)" src="trash.png" class="clear-button">
                        </div>
                    </div>
                </div>
                <div v-show="sharedItems.length != 0" class="flex-center-end mr-1-5">
                    <button @click="addSharedItem" class="button">
                        Add a shared item
                    </button>
                </div>
            </div>
            <div class="flex-center-between mb-half mr-1-5">
                <div class="mr-half text-bold">Subtotal:</div>
                <div>$ <input v-model="subtotal" type="number" min="0" step="0.01"></div>
            </div>
            <div class="flex-center-between mb-half mr-1-5">
                <div class="mr-half text-bold">Total:</div>
                <div>$ <input v-model="total" type="number" :min="subtotal" step="0.01"></div>
            </div>
            <div class="flex-center-end mr-1-5">
                <button @click="calculate" class="button">Calculate</button>
            </div>
            <div v-show="showCalculation" ref="results">
                <h4 class="mb-half text-bold">Results:</h4>
                <div v-for="vagabond in this.vagabonds">
                    {{ vagabond.name ? vagabond.name : `Vagabond ${vagabond.index}` }}
                    owes {{ recipient ? recipient : "The Lender" }}
                    ${{ vagabond.totalAmountDue }}
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            showExplanation: false,
            recipient: "",
            vagabonds: [],
            sharedItems: [],
            subtotal: 0,
            total: 0,
            sharedItemSplit: 0,
            fees: 0,
            showCalculation: false,
        }
    },
    methods: {
        addVagabond() {
            this.showCalculation = false
            this.vagabonds.push({
                index: this.vagabonds.length + 1,
                name: "",
                items: [{
                    name: "",
                    price: null,
                }],
                baseAmountDue: 0,
                feesPortion: 0,
                totalAmountDue: 0,
            })
        },
        removeVagabond(index) {
            this.showCalculation = false
            this.vagabonds.splice(index, 1)
        },
        addItem(vagabond) {
            this.showCalculation = false
            vagabond.items.push({
                name: "",
                price: null,
            })
        },
        removeItem(vagabond_index, item_index) {
            this.showCalculation = false
            this.vagabonds[vagabond_index].items.splice(item_index, 1)
        },
        addSharedItem() {
            this.showCalculation = false
            this.sharedItems.push({
                name: "",
                price: null,
            })
        },
        removeSharedItem(index) {
            this.showCalculation = false
            this.sharedItems.splice(index, 1)
        },
        calculate() {
            if (!this.total || !this.subtotal) {
                return
            }
            // - Calculate fees
            this.fees = this.total - this.subtotal

            // - Calculate shared item sum
            let sharedItemSum = 0
            for (let sharedItem of this.sharedItems) {
                sharedItemSum += sharedItem.price
            }
            this.sharedItemSplit = sharedItemSum / (this.vagabonds.length + 1)

            // - Calculate vagabond amountDue and portion of fees
            for (let vagabond of this.vagabonds) {
                vagabond.baseAmountDue = 0
                for (let item of vagabond.items) {
                    vagabond.baseAmountDue += item.price
                }

                vagabond.feesPortion = ((vagabond.baseAmountDue + this.sharedItemSplit) / this.subtotal) * this.fees
                vagabond.totalAmountDue = vagabond.baseAmountDue + vagabond.feesPortion + this.sharedItemSplit
            }
            this.showCalculation = true
            window.scrollTo(0, this.$refs.results.offsetTop)
        },
    },
}
</script>