<template>
    <b-container>
        <b-card no-body>
            <b-card-header>
                <span class="h4 mr-3">Block</span>
                <template v-if="block">
                    #{{block.number}}
                    <b-badge v-if="!block.isTrunk" class="ml-3" variant="warning">Branch</b-badge>
                </template>
            </b-card-header>
            <b-card-body>
                <template v-if="block">
                    <b-row>
                        <b-col lg="2">
                            <strong>ID</strong>
                        </b-col>
                        <b-col lg="10" class="d-flex">
                            <span class="text-monospace text-truncate">{{block.id}}</span>
                            <Copy :value="block.id" class="ml-2" />
                        </b-col>
                    </b-row>
                    <hr />
                    <b-row>
                        <b-col lg="2">
                            <strong>Size</strong>
                        </b-col>
                        <b-col lg="10">{{block.size|locale}} B</b-col>
                    </b-row>
                    <hr />
                    <b-row>
                        <b-col lg="2">
                            <strong>Timestamp</strong>
                        </b-col>
                        <b-col lg="10">{{block.timestamp | date}}</b-col>
                    </b-row>
                    <hr />
                    <b-row>
                        <b-col lg="2">
                            <strong>Gas Used</strong>
                        </b-col>
                        <b-col lg="10">
                            {{block.gasUsed | locale}}/{{block.gasLimit | locale}}
                            <b-button
                                v-if="txs.length"
                                size="sm"
                                variant="primary"
                                class="ml-3 py-0"
                                v-b-toggle.txs
                            >{{txs.length}} {{txs.length>1?'transactions': 'transaction'}}</b-button>
                        </b-col>
                    </b-row>
                    <b-collapse v-if="txs.length" id="txs">
                        <ol start="0" class="text-monospace mb-0 mt-3 small">
                            <li v-for="(tx, i) in txs" :key="i" class="mt-2">
                                <router-link
                                    :to="{name: 'tx', params:{id: tx}}"
                                    style="display: inline-block;"
                                    class="text-truncate w-100 align-top"
                                >{{tx}}</router-link>
                            </li>
                        </ol>
                    </b-collapse>
                    <hr />

                    <b-row>
                        <b-col lg="2">
                            <strong>Parent</strong>
                        </b-col>
                        <b-col lg="10" class="text-truncate">
                            <router-link
                                v-if="block.number>0"
                                class="text-monospace"
                                :to="{name:'block', params: {id: block.parentID}}"
                            >#{{block.number-1}}</router-link>
                            <span v-else>N/A</span>
                        </b-col>
                    </b-row>
                    <hr />
                    <b-row>
                        <b-col lg="2">
                            <strong>Total Score</strong>
                        </b-col>
                        <b-col lg="10">{{block.totalScore | locale}}</b-col>
                    </b-row>
                    <hr />
                    <b-row>
                        <b-col lg="2">
                            <strong>Features</strong>
                        </b-col>
                        <b-col lg="10">
                            <b-badge v-if="vip191Supported" variant="info">VIP-191</b-badge>
                            <div v-else>None</div>
                        </b-col>
                    </b-row>
                    <hr />
                    <b-row>
                        <b-col lg="2">
                            <strong>Signer</strong>
                        </b-col>
                        <b-col lg="10">
                            <AccountLink :address="block.signer" />
                        </b-col>
                    </b-row>
                    <hr />
                    <b-row>
                        <b-col lg="2">
                            <strong>Beneficiary</strong>
                        </b-col>
                        <b-col lg="10">
                            <AccountLink :address="block.beneficiary" />
                        </b-col>
                    </b-row>
                    <hr />
                    <b-row>
                        <b-col lg="2">
                            <strong>State Root</strong>
                        </b-col>
                        <b-col lg="10" class="text-monospace text-truncate">{{block.stateRoot}}</b-col>
                    </b-row>
                    <hr />
                    <b-row>
                        <b-col lg="2">
                            <strong>Txs Root</strong>
                        </b-col>
                        <b-col lg="10" class="text-monospace text-truncate">{{block.txsRoot}}</b-col>
                    </b-row>
                    <hr />
                    <b-row>
                        <b-col lg="2">
                            <strong>Receipts Root</strong>
                        </b-col>
                        <b-col lg="10" class="text-monospace text-truncate">{{block.receiptsRoot}}</b-col>
                    </b-row>
                </template>
                <template v-else>
                    <div v-if="error" class="text-center">
                        <p>Oops</p>
                        <p class="text-warning">Error: {{error.message}}</p>
                        <b-button size="sm" @click="reload">Reload</b-button>
                    </div>
                    <Loading v-else class="my-3" />
                </template>
            </b-card-body>
        </b-card>
    </b-container>
</template>
<script lang="ts">
import { Vue, Component } from 'vue-property-decorator'

@Component({ name: 'Block' })
export default class Block extends Vue {
    private block: Connex.Thor.Block | null = null
    private error: Error | null = null

    private id = ''
    get txs() { return this.block!.transactions }
    // tslint:disable-next-line:no-bitwise
    get vip191Supported() { return !!(((this.block as any).txsFeatures || 0) & 1) }

    private async reload() {
        this.block = null
        this.error = null

        try {
            const block = await this.$connex.thor.block(this.id).get()
            if (!block) {
                this.error = new Error('block not found')
            } else {
                this.block = block
            }
        } catch (err) {
            this.error = err
        }
    }

    private created() {
        this.$ga.page('/insight/block')
        this.id = this.$route.params.id
        this.reload()
    }
}
</script>
