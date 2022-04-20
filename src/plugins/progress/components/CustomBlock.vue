<script>
import Plugin from '../index';
import { shorten } from '@/helpers/utils';
import { useIntl } from '@/composables/useIntl';
import { useWeb3 } from '@/composables/useWeb3';

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
      showEditPanel: false,
      plugin: new Plugin(),
      steps: [
        {
          id: 1,
          text: 'Voting',
          status: 'active'
        },
        {
          id: 2,
          text: 'Passed',
          status: 'pending'
        }
      ]
    };
  },
  computed: {
    isOwner() {
      return web3Account.value === this.proposal.author;
    },
    isComplete() {
      return this.proposal.state !== 'active';
    }
  },
  methods: {
    async getActiveSteps() {
      //TODO: Call api with proposalID to retrieve current activeStep
      const response = fetch(
        'https://jissr670k3.execute-api.us-east-1.amazonaws.com/dev/proposal/1'
      ).then(response => response.json());
      return true;
      // https://1f4w55dofg.execute-api.us-east-1.amazonaws.com/default/getProposalStep?proposalId=1
    },
    async incrementStep() {
      // const auth = getInstance();
      // let sig;

      // const msg = {
      //   author: web3Account.value,
      //   msg: this.activeStep++,
      //   proposal_id: this.proposal.proposalId
      // };
      // sig = await signMessage(
      //   auth.web3,
      //   JSON.stringify(msg),
      //   web3Account.value
      // );

      // TODO: show confirmation modal
      // TODO: call API to increment activeStep
      this.activeStep++;
    },
    async createNewStep() {
      // TODO: call API to create a new step for this proposal
    },
    async setStepComplete() {
      //Call API to set step status to complete
      const response = fetch(
        'https://jissr670k3.execute-api.us-east-1.amazonaws.com/dev/proposal/1'
      ).then(response => response.json());
    }
  }
};
</script>

<template>
  <BaseBlock title="Progress" :loading="!loaded">
    <div class="flex flex-col">
      <div class="stepper-wrapper">
        <div
          v-for="step in steps"
          :key="step.id"
          v-bind:class="{
            active: step.status === 'active',
            completed: step.status === 'complete'
          }"
          class="stepper-item"
        >
          <div class="step-counter">{{ step.id }}</div>
          <div class="step-name">{{ step.text }}</div>
        </div>
      </div>
      <div class="w-100">
        <BaseButton
          @click="getActiveSteps()"
          class="!bg-primary !text-white w-full"
        >
          Get Step
        </BaseButton>
        <BaseButton
          @click="showEditPanel = true"
          class="!bg-primary !text-white w-full"
          v-if="(isAdmin || isOwner) && !isComplete"
        >
          Edit
        </BaseButton>
        <BaseBlock v-if="showEditPanel">
          <div v-for="step in steps" :key="step.id">
            {{ step.text }} Status: {{ step.status }}
            <BaseButton
              class="!bg-primary !text-white"
              v-if="step.status === 'active'"
              @click="setStepComplete(step.id)"
            >
              Complete
            </BaseButton>
          </div>
          Step Name
          <BaseButton class="w-full">
            <input class="input w-full text-center" />
          </BaseButton>
          <BaseButton
            @click="createNewStep()"
            class="!bg-primary !text-white w-full"
            v-if="isAdmin || isOwner"
          >
            Add Step
          </BaseButton>
        </BaseBlock>
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
  color: white;
  z-index: 5;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: #565656;
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
