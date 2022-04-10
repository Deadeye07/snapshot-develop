<script>
import getProvider from '@snapshot-labs/snapshot.js/src/utils/provider';
import Plugin from '../index';
import { shorten } from '@/helpers/utils';
import { useIntl } from '@/composables/useIntl';
import { useWeb3 } from '@/composables/useWeb3';
import { ref, onMounted, computed, watch } from 'vue';
import { signMessage } from '@snapshot-labs/snapshot.js/src/utils/web3';
import { getInstance } from '@snapshot-labs/lock/plugins/vue3';

const { formatCompactNumber, formatPercentNumber } = useIntl();
const { web3Account } = useWeb3();

export default {
  setup() {
    return { shorten, formatCompactNumber, formatPercentNumber };
  },
  props: ['space', 'proposal', 'results', 'loaded', 'strategies'],
  data() {
    return {
      loading: false,
      plugin: new Plugin(),
      activeStep: 0
    };
  },
  computed: {
    totalScore() {
      const basicCount = this.space.plugins?.quorum?.basicCount;
      if (basicCount && this.proposal.type === 'basic') {
        return this.results.resultsByVoteBalance
          .filter((score, i) => basicCount.includes(i))
          .reduce((a, b) => a + b, 0);
      }
      return this.results.resultsByVoteBalance.reduce((a, b) => a + b, 0);
    },
    isOwner() {
      return web3Account.value === this.proposal.author;
    }
  },
  methods: {
    async incrementStep() {
      const auth = getInstance();
      let sig;

      const msg = {
        author: web3Account.value,
        msg: this.activeStep++,
        proposal_id: this.proposal.proposalId
      };
      sig = await signMessage(
        auth.web3,
        JSON.stringify(msg),
        web3Account.value
      );
      const res = await this.postData(
        `https://uia5m1.deta.dev/add`,
        {
          address: web3Account.value,
          msg: this.activeStep,
          sig
        },
        //token ? { authorization: token } : null
      );
    },

    async postData(url = '', data = {}, authorization) {
      // Default options are marked with *

      const response = await fetch(url, {
        method: 'POST',
        body: JSON.stringify(data),
        headers: {
          'Content-type': 'application/json;charset=UTF-8',
          ...authorization
        }
      });
      return response.json(); // parses JSON response into native JavaScript objects
    }
  }
};
</script>

<template>
  <BaseBlock title="Progress" :loading="!loaded">
    <BaseButton
      @click="incrementStep()"
      class="!bg-primary !text-white float-right"
      v-if="isAdmin || isOwner"
    >
      Update
    </BaseButton>
    <div class="stepper-wrapper">
      <div
        v-bind:class="{ active: activeStep === 1, completed: activeStep > 1 }"
        class="stepper-item"
      >
        <div class="step-counter">1</div>
        <div class="step-name">Voting</div>
      </div>
      <div
        class="stepper-item"
        v-bind:class="{ active: activeStep === 2, completed: activeStep > 2 }"
      >
        <div class="step-counter">2</div>
        <div class="step-name">Passed</div>
      </div>
      <div
        class="stepper-item"
        v-bind:class="{ active: activeStep === 3, completed: activeStep > 3 }"
      >
        <div class="step-counter">3</div>
        <div class="step-name">Purchase of Tokens</div>
      </div>
      <div
        class="stepper-item"
        v-bind:class="{ active: activeStep === 4, completed: activeStep > 4 }"
      >
        <div class="step-counter">4</div>
        <div class="step-name">Completed</div>
      </div>
    </div>
  </BaseBlock>
</template>
<style>
.stepper-wrapper {
  margin-top: auto;
  display: flex;
  justify-content: space-between;
  margin-bottom: 20px;
}
.stepper-item {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  flex: 1;

  @media (max-width: 768px) {
    font-size: 12px;
  }
}

.stepper-item::before {
  position: absolute;
  content: '';
  border-bottom: 2px solid #ccc;
  width: 100%;
  top: 20px;
  left: -50%;
  z-index: 2;
}

.stepper-item::after {
  position: absolute;
  content: '';
  border-bottom: 2px solid #ccc;
  width: 100%;
  top: 20px;
  left: 50%;
  z-index: 2;
}

.stepper-item .step-counter {
  position: relative;
  z-index: 5;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: #ccc;
  margin-bottom: 6px;
}

.stepper-item.active {
  font-weight: bold;
}

.stepper-item.completed .step-counter {
  background-color: #4bb543;
}

.stepper-item.completed::after {
  position: absolute;
  content: '';
  border-bottom: 2px solid #4bb543;
  width: 100%;
  top: 20px;
  left: 50%;
  z-index: 3;
}

.stepper-item:first-child::before {
  content: none;
}
.stepper-item:last-child::after {
  content: none;
}
.step-name {
  text-align: center;
}
</style>
