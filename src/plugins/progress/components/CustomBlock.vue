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
      addIsLoading: false,
      completeIsLoading: false,
      showEditPanel: false,
      newStepDescription: '',
      plugin: new Plugin(),
      steps: []
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
  beforeMount() {
    this.getActiveSteps();
  },
  methods: {
    async getActiveSteps() {
      if (this.isComplete) {
        const apiUrl =
          'https://jissr670k3.execute-api.us-east-1.amazonaws.com/dev/proposal/' +
          this.proposal.id;
        fetch(apiUrl).then(response =>
          response
            .json()
            .then(
              data =>
                (this.steps = data.Items.sort((a, b) =>
                  a.index > b.index ? 1 : -1
                ))
            )
        );
      }
    },
    async createNewStep() {
      if (this.newStepDescription !== '') {
        this.addIsLoading = true;
        const apiUrl =
          'https://jissr670k3.execute-api.us-east-1.amazonaws.com/dev/proposal/' +
          this.proposal.id;
        const requestOptions = {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ description: this.newStepDescription })
        };
        fetch(apiUrl, requestOptions).then(response =>
          response.json().then(response => {
            this.getActiveSteps();
            this.addIsLoading = false;
            this.newStepDescription = '';
          })
        );
      }
    },
    async setStepComplete(step) {
      this.completeIsLoading = true;
      const requestOptions = {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ proposalId: step.proposalId })
      };
      const response = fetch(
        'https://jissr670k3.execute-api.us-east-1.amazonaws.com/dev/proposal/' +
          step.id,
        requestOptions
      ).then(response => {
        this.getActiveSteps();
        this.completeIsLoading = false;
      });
    },
    async deleteStep(step) {
      const apiUrl =
        'https://jissr670k3.execute-api.us-east-1.amazonaws.com/dev/proposal/' +
        step.id +
        '?proposalId=' +
        this.proposal.id;
      const requestOptions = {
        method: 'DELETE'
      };
      fetch(apiUrl, requestOptions).then(() => {
        this.getActiveSteps();
      });
    }
  }
};
</script>

<template>
  <BaseBlock title="Progress" :loading="!loaded">
    <div class="flex flex-row-reverse mb-4">
      <i
        @click="showEditPanel = !showEditPanel"
        v-if="(isAdmin || isOwner) && isComplete"
        class="edit-icon iconfont icongear hover:text-skin-link cursor-pointer relative"
        style="font-size: 25px; line-height: 25px"
      ></i>
    </div>
    <div class="flex flex-col">
      <div class="stepper-wrapper">
        <div class="stepper-item completed">
          <div class="step-counter">1</div>
          <div class="step-name">Voting</div>
        </div>
        <div
          v-bind:class="{
            active: !isComplete,
            completed: isComplete
          }"
          class="stepper-item"
        >
          <div class="step-counter">2</div>
          <div class="step-name">Voting Complete</div>
        </div>
        <div
          v-for="step in steps"
          :key="step.id"
          v-bind:class="{
            active: step.stepStatus === 'active',
            completed: step.stepStatus === 'complete'
          }"
          class="stepper-item"
        >
          <div class="step-counter">{{ step.index + 2 }}</div>
          <div class="step-name">{{ step.description }}</div>
        </div>
      </div>
      <div class="w-100">
        <BaseBlock class="mt-4" v-if="showEditPanel">
          <div class="font-bold mb-4">
            <span class="text-xl">Update</span>
          </div>
          <div class="flex text-xl mb-2">
            <div class="w-1/5 text-center">Step</div>
            <div class="w-1/2"></div>
            <div class="w-1/4">Completed</div>
          </div>
          <div class="mb-4" v-for="step in steps" :key="step.id">
            <div class="flex mb-4">
              <div class="w-1/5">
                <div
                  v-bind:class="{
                    active: step.stepStatus === 'active',
                    completed: step.stepStatus === 'complete'
                  }"
                  class="stepper-item"
                >
                  <div class="step-counter">{{ step.index + 2 }}</div>
                </div>
              </div>
              <div class="w-1/2">
                {{ step.description }}
              </div>

              <div class="w-1/4">
                <a
                  v-if="step.stepStatus === 'active'"
                  @click="setStepComplete(step)"
                  style="min-height: 41px; min-width: 41px"
                  class="!bg-primary !text-white button button-small"
                >
                  <i
                    v-if="!completeIsLoading"
                    class="iconfont iconcheck"
                    style="font-size: 25px; line-height: 25px"
                  ></i>
                  <div v-if="completeIsLoading" class="spinner"></div>

                  <span v-if="!completeIsLoading" class="label-hidden"
                    >Complete</span
                  >
                </a>
                <a
                  v-if="step.stepStatus !== 'active'"
                  class="pointer-events-none !bg-primary !text-green button button-small"
                >
                  <i
                    class="iconfont iconcheck"
                    style="font-size: 25px; line-height: 25px"
                  ></i>
                </a>
              </div>
              <div class="text-right pt-2">
                <i
                  v-if="step.stepStatus === 'active'"
                  class="iconfont iconclose cursor-pointer"
                  @click="deleteStep(step)"
                  style="font-size: 25px; line-height: 25px"
                ></i>
              </div>
            </div>
            <hr />
          </div>
          <br /><br />
          <span class="text-xl">New</span>
          <BaseButton class="w-full mb-2 mt-2">
            <input
              placeholder="Add new step"
              v-model="newStepDescription"
              class="input w-full text-center font-bold"
            />
          </BaseButton>
          <BaseButton
            @click="createNewStep()"
            class="!bg-primary !text-white w-full"
            v-if="isAdmin || isOwner"
          >
            <span v-if="!addIsLoading">Add</span>
            <div v-if="addIsLoading" class="relative spinner"></div>
          </BaseButton>
        </BaseBlock>
      </div>
    </div>
  </BaseBlock>
</template>
<style>
a {
  text-decoration: none;
}
a.button .iconfont {
  font-size: 24px;
  transition: 0.4s;
}
a.button:hover .label-hidden {
  max-width: 200px;
  margin-left: 8px;
  opacity: 1;
}
a.button .label-hidden {
  max-width: 0;
  opacity: 0;
  white-space: nowrap;
  transition: 0.4s;
}
a.button {
  position: relative;
  display: inline-flex;
  align-items: center;
  justify-content: flex-start;
  overflow: hidden;
  padding: 8px;
  border-radius: 8px;
  transition: 0.2s;
}
.edit-icon {
  top: -65px;
}
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

.step-counter {
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

@keyframes spinner {
  to {
    transform: rotate(360deg);
  }
}

.spinner:before {
  content: '';
  box-sizing: border-box;
  position: absolute;
  top: 50%;
  left: 50%;
  width: 20px;
  height: 20px;
  margin-top: -10px;
  margin-left: -10px;
  border-radius: 50%;
  border: 2px solid #f6f;
  border-top-color: #0e0;
  border-right-color: #0dd;
  border-bottom-color: #f90;
  animation: spinner 0.6s linear infinite;
}
</style>
