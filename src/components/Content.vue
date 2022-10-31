<script setup>
import { ref, onMounted } from "vue";
import { useLoadingBar, useMessage, useNotification } from "naive-ui";

import { OracleAgeProof_ as ZkProofOfAge_ } from "zkapp-proof-of-age-oracle";

import {
  Field,
  PrivateKey,
  PublicKey,
  Mina,
  isReady,
  fetchAccount,
  setGraphqlEndpoint,
  Poseidon,
} from "snarkyjs";

// ui components
const loadingBar = useLoadingBar();
const message = useMessage();
const notification = useNotification();
const loadingSnarkyJs = ref(true);

// zk-app
const zkAppAddress = ref("");
const zkAppState = ref("");
const zkApp = ref({});
const transaction = ref({});
const userOracleID = ref();
const userAge = ref();
const proofOfAge = ref();

// keys
const publicKey_ = ref("");
const privateKey_ = ref("");

const generateNewKeys = () => {
  const privateKey__ = PrivateKey.random();
  privateKey_.value = privateKey__.toBase58();
  publicKey_.value = privateKey__.toPublicKey().toBase58();
};

// this is the steps vals
const stepsStatus = ref({
  currentStatus: "process",
  current: 1,
});

const steps = ref({
  1: {
    isFinished: false,
    isLoading: false,
  },
  2: {
    isFinished: false,
    isLoading: false,
  },
  3: {
    isFinished: false,
    isLoading: false,
  },
  4: {
    isFinished: false,
    isLoading: false,
  },
  5: {
    isFinished: false,
    isLoading: false,
  },
  6: {
    isFinished: false,
    isLoading: false,
  },
});

const sleep = (ms) => {
  return new Promise((resolve) => setTimeout(resolve, ms));
};

onMounted(async () => {
  let msg = message.create("Loading", { type: "success", duration: 10000 });
  await sleep(1500);
  await isReady;
  loadingSnarkyJs.value = false;

  // create Berkeley connection
  const graphqlEndpoint = "https://proxy.berkeley.minaexplorer.com/graphql";
  setGraphqlEndpoint(graphqlEndpoint);
  let Berkeley = Mina.BerkeleyQANet(graphqlEndpoint);
  Mina.setActiveInstance(Berkeley);
  msg.content = "Connected to Berkeley!";
});

const checkAccountBalance = async () => {
  loadingBar.start();
  steps.value[1].isLoading = true;

  if (!publicKey_.value) {
    message.warning("Generate or input a key pair!");
    loadingBar.error();
    steps.value[1].isLoading = false;
    return;
  } else {
    console.log("try to fetch an account", publicKey_.value);

    let { account, error } = await fetchAccount({
      publicKey: PublicKey.fromBase58(publicKey_.value),
    });

    console.log("account", account);
    console.log("error", JSON.stringify(error, null, 2));

    if (account) {
      message.success(`Account balance: ${account.balance}`);
    } else {
      message.warning("No balance in the account?");
      loadingBar.error();
      steps.value[1].isLoading = false;
      return;
    }
  }
  steps.value[1].isLoading = false;
  steps.value[1].isFinished = true;
  stepsStatus.value.current = 2;
  loadingBar.finish();
};

const compileZkApp = async (zkAppAddress) => {
  loadingBar.start();
  steps.value[2].isLoading = true;
  await sleep(500);
  message.warning("This may take a while. Please be patient.");

  // compile the zkapp
  console.log("Before compiling");
  await sleep(1500);
  console.log("Compiling smart contract...", zkAppAddress);
  await ZkProofOfAge_.compile();
  console.log(ZkProofOfAge_);
  console.log("Done.");
  steps.value[2].isLoading = false;
  steps.value[2].isFinished = true;
  stepsStatus.value.current = 3;
  loadingBar.finish();
};

const getZkAppState = async (zkAppAddress) => {
  loadingBar.start();
  steps.value[3].isLoading = true;

  console.log(
    "PUBLICKEY: ",
    zkAppAddress.value,
    PublicKey.fromBase58(zkAppAddress.value)
  );

  let { account, error } = await fetchAccount({
    publicKey: PublicKey.fromBase58(zkAppAddress.value),
  });

  console.log("account", JSON.stringify(account, null, 2));
  console.log("error", JSON.stringify(error, null, 2));

  if (error) {
    steps.value[3].isLoading = false;
    steps.value[3].isFinished = true;
    stepsStatus.value.current = 2;
    loadingBar.error();
    return;
  }

  // create the zkapp object
  try {
    console.log("trying to create zkapp object");
    zkApp.value = new ZkProofOfAge_(PublicKey.fromBase58(zkAppAddress));
    let value = zkApp.value.value.get();
    zkAppState.value = value;

    console.log(`Found deployed zkapp with state ${value.toBase58()}`);
  } catch (error) {
    steps.value[3].isLoading = false;
    steps.value[3].isFinished = true;
    stepsStatus.value.current = 3;
    loadingBar.error();
    return;
  }
  steps.value[3].isLoading = false;
  steps.value[3].isFinished = true;
  stepsStatus.value.current = 4;
  loadingBar.finish();
};

const computeAnswer = async (userOracleID, userAge) => {
  let answer = Poseidon.hash([userOracleID]);
  for (let i = 0; i < userAge; ++i) {
    answer = Poseidon.hash([answer]);
  }
  return answer;
};

const createTransaction = async () => {
  loadingBar.start();
  steps.value[4].isLoading = true;

  console.log("Creating transaction");

  try {
    let feePayerKey = PrivateKey.fromBase58(privateKey_.value);
    let answer = await computeAnswer(userOracleID, userAge);
    transaction.value = await Mina.transaction(
      { feePayerKey, fee: "100_000_000" },
      () => {
        zkApp.value.giveAnswer(answer, PublicKey.fromBase58(publicKey_.value));
      }
    );
    message.success("You seem to have a valid proof and ...", {
      duration: 10000,
    });
    message.success(
      "You have successfully generated a transaction. It can be sent once the proof is generated in the next step.",
      { duration: 10000 }
    );
  } catch (error) {
    message.error("Error, could not create transaction", { duration: 10000 });
    steps.value[4].isLoading = false;
    steps.value[4].isFinished = true;
    stepsStatus.value.current = 4;
    loadingBar.error();
    return;
  }

  steps.value[4].isLoading = false;
  steps.value[4].isFinished = true;
  stepsStatus.value.current = 5;

  loadingBar.finish();
};

const createProof = async () => {
  message.warning("This may take a while. Please be patient.");

  loadingBar.start();
  steps.value[5].isLoading = true;

  await sleep(500);

  try {
    await transaction.value.prove();
    console.log("Transaction proof:");
    console.log(
      transaction.value.transaction.accountUpdates[0].authorization.proof
    );
  } catch (error) {
    steps.value[5].isLoading = false;
    steps.value[5].isFinished = true;
    stepsStatus.value.current = 5;
    loadingBar.error();
    return;
  }

  notification.create({
    title: "You successfully generated the proof!",
    content:
      "Here is what it looks like: \n\n" +
      transaction.value.transaction.accountUpdates[0].authorization.proof.slice(
        0,
        128
      ) +
      " ...",
  });

  steps.value[5].isLoading = false;
  steps.value[5].isFinished = true;
  stepsStatus.value.current = 6;
  loadingBar.finish();
};

const broadcastTransaction = async () => {
  loadingBar.start();
  steps.value[6].isLoading = true;

  // send the transaction to the graphql endpoint
  console.log("Sending the transaction...");
  try {
    let sendZkapp = await transaction.value.send();
    let txHash = await sendZkapp.hash();

    console.log("Transaction hash:");
    console.log(txHash);

    message.success(
      "Your transaction is sent. The state of the smart contract will be updated as soon as the transaction is included in the next block!",
      { duration: 10000 }
    );
    notification.create({
      title: "Transaction information",
      content:
        "Your transaction's  hash: " +
        txHash +
        "\n\n" +
        "You can check its status on https://berkeley.minaexplorer.com/",
    });
  } catch (error) {
    steps.value[6].isLoading = false;
    steps.value[6].isFinished = true;
    stepsStatus.value.current = 6;
    loadingBar.error();
    return;
  }
  steps.value[6].isLoading = false;
  steps.value[6].isFinished = true;
  stepsStatus.value.current = 6;
  loadingBar.finish();
};
</script>

<template>
  <div style="padding: 2px; max-width: 580px">
    <n-modal v-model:show="loadingSnarkyJs">
      <n-card style="max-width: 150px">
        <n-space vertical style="text-align: center">
          Loading snarkyJs
          <n-spin size="large" />
        </n-space>
      </n-card>
    </n-modal>

    <n-space vertical>
      <n-h2
        >Before we begin, make sure you have an account with a balance.
      </n-h2>
      <n-text>
        Or, if you do not have an account, you can click the button below to
        generate a new pair of keys. Use the
        <a href="https://berkeley.minaexplorer.com/faucet">faucet link</a> to
        request mina.
      </n-text>
      <n-button @click="generateNewKeys()">Generate new pair of keys</n-button>
      <br />
      <n-input-group>
        <n-input-group-label>Public Key</n-input-group-label>
        <n-input v-model:value="publicKey_" />
      </n-input-group>
      <n-input-group>
        <n-input-group-label>Private Key</n-input-group-label>
        <n-input v-model:value="privateKey_" />
      </n-input-group>
      <br />
      <n-h2> The oracle must have deployed a zkApp for you. </n-h2>
      <n-text>
        Please enter the public key of this address. You will only be able to
        interact with this zkApp if you are the person the Oracle knows.</n-text
      >
      <br />
      <n-input-group>
        <n-input-group-label>zkApp Public Key</n-input-group-label>
        <n-input v-model:value="zkAppAddress" />
      </n-input-group>
    </n-space>
    <br /><br />
    <n-divider />
    <br /><br />
    <n-space vertical>
      <n-h2>Follow these steps to execute the smart contract:</n-h2>
      <n-steps
        vertical
        :current="stepsStatus.current"
        :status="stepsStatus.currentStatus"
      >
        <n-step title="Check if the account has funds">
          <n-space vertical>
            The transaction will cost 0.1 mina.
            <n-button
              @click="checkAccountBalance()"
              :loading="steps[1].isLoading"
              >Check</n-button
            >
          </n-space>
        </n-step>
        <n-step title="Compile the smart contract we will interact with">
          <n-space vertical>
            <div>
              Check out the smart contract
              <a href="https://github.com/">here</a>.
            </div>
            <n-button
              @click="compileZkApp(zkAppAddress)"
              :loading="steps[2].isLoading"
              >Compile</n-button
            >
          </n-space>
        </n-step>
        <n-step title="Check the smart contract state on-chain">
          <n-space vertical>
            The state is a public key. It will be updated with your public key
            if you prove your age successfully, showing that the address belongs
            to a person above 18.
            <n-button
              @click="getZkAppState(zkAppAddress)"
              :loading="steps[3].isLoading"
              >Check</n-button
            >
            <n-tag :size="'large'" style="padding: 30px" :bordered="false">
              <div v-if="steps[3].isLoading">
                <n-spin size="small" />
              </div>
              <div else>
                <n-text depth="3">
                  {{ zkAppState ? zkAppState : "" }}
                </n-text>
              </div>
            </n-tag>
            You will update this state if you are older than 18.
          </n-space>
        </n-step>
        <n-step title="Call the smart contract method">
          <n-space vertical>
            <div>
              In order to this, please input your Oracle ID and your age. No
              worries, none of this information is shared with anyone. They are
              only used for computations done
              <b>locally in your browser</b>.
            </div>
            <n-input-group>
              <n-input-group-label>Your Oracle ID</n-input-group-label>
              <n-input v-model:value="userOracleID" />
            </n-input-group>
            <n-input-group>
              <n-input-group-label>Your Age</n-input-group-label>
              <n-input v-model:value="userAge" />
            </n-input-group>
            <n-button @click="createTransaction()" :loading="steps[4].isLoading"
              >Call</n-button
            >
          </n-space>
        </n-step>
        <n-step title="Create the zero knowledge proof">
          <n-space vertical>
            The proof will confirm that all the things that happened in your
            browser while interacting with the smart contract are legitimate.
            <n-button @click="createProof()" :loading="steps[5].isLoading"
              >Create</n-button
            >
          </n-space>
        </n-step>
        <n-step title="Broadcast the transaction to the network">
          <n-button
            @click="broadcastTransaction()"
            :loading="steps[6].isLoading"
            >Broadcast</n-button
          >
        </n-step>
      </n-steps>
    </n-space>
    <br /><br />
  </div>
</template>

<style scoped></style>
