<script lang="ts">
  import MessageBouble from "../MessageBouble.svelte";
  import { onMount } from 'svelte';
  import axios from "axios";

  axios.defaults.baseURL = 'http://localhost:6001';
  // axios.defaults.baseURL = 'https://88db-35-208-224-244.ngrok-free.app';
  axios.defaults.headers.common['Access-Control-Allow-Origin'] = "*"
  axios.defaults.headers.common['Content-Type'] = "application/x-www-form-urlencoded"
  axios.defaults.headers.common['ngrok-skip-browser-warning'] = 'true'
  let activeChat = '';
  let activeChatName = '';
  let activeChatAgent = '';
  let messages:any = [];
  let conversations = [];
  let respTimes = [];
  let selectedModel = "";
  let chatName = "";
  export let showModal;
  let dialog;
  $: if (dialog && showModal) dialog.showModal();
  let userInput = '';

  function uuidv4() {
      return ([1e7]+-1e3+-4e3+-8e3+-1e11).replace(/[018]/g, c =>
          (c ^ crypto.getRandomValues(new Uint8Array(1))[0] & 15 >> c / 4).toString(16)
      );
  }

  onMount(() => {
      loadConversationsHistory()
  })

  const loadConversationsHistory = ():void => {
    const conversationHistory = JSON.parse(localStorage.getItem("conversations_history")) || [];
    conversations = [...conversationHistory];
  }

  const loadMessagesFromBackend = async (chat_uuid) => {
        const res = await axios.get(`/get_messages?conv_uuid=${chat_uuid}`, {
            headers: {
                "Content-Type": "application/x-www-form-urlencoded",
                "Access-Control-Allow-Origin": "*",
                "Access-Control-Allow-Headers": "*"
            }
        });

        if (res.status === 200) {
            return res.data
        } else {
            console.log(res.request.error)
        }

  }

  const createNewConversation = (): void => {
      let conversationUuid = uuidv4();
      conversations = [...conversations, {uuid: conversationUuid, model: selectedModel, name: chatName}]
      localStorage.setItem("conversations_history", JSON.stringify([...conversations]))
      activeChatAgent = selectedModel;
      activeChat = conversationUuid;
      activeChatName = chatName;
      selectActiveChat({uuid: conversationUuid, model: selectedModel, name: chatName})
      selectedModel = "";
      chatName = "";
      dialog.close();

  }



  const selectActiveChat = async (chat): Promise<void> => {
      messages = [];
      respTimes = [];
      activeChat = chat.uuid;
      activeChatName = chat.name;
      activeChatAgent = chat.model;
      const msg_from_db = await loadMessagesFromBackend(chat.uuid);
      if (msg_from_db !== 'No data found for given conversation UUID') {
          messages = msg_from_db[0].messages;
          respTimes = msg_from_db[0].resp_time;
      }
  }

  const sendMessage = async (): Promise<void> => {
      messages = [...messages, {"role": "user", "content": userInput, "request_time": 0}];
      respTimes = [...respTimes, 0];
      const res = await axios.post('/chat', {
          "conv_uuid": activeChat,
          "conv_agent": activeChatAgent,
          "conv_name": activeChatName,
          "body": userInput
      }, {
          headers: {
              'Content-Type': 'application/x-www-form-urlencoded'
          }
        }
      )
      userInput = "";
      const msg = res.data
      console.log(msg)
      messages = [...messages, {"role": msg.data.role, "content": msg.data.content}];
      respTimes = [...respTimes, msg.resp_time]
  }

</script>

<div class="content-container">
    <div class="conversation_history">
        <h3>Chat history</h3>
        <button class="new-chat-btn" on:click={() => showModal = true}>New chat</button>
        <ul>
            {#each conversations as chat, i (chat.uuid)}
                <li on:click={() => selectActiveChat(chat)} class="{`${chat.uuid === activeChat ? 'active_chat' : ''}`}"><span>{i + 1}. </span>{chat.name || chat.uuid}</li>
            {/each}
        </ul>
    </div>
    <div class="conversation">
        <h1>ChatGPT</h1>
        <h3>Active chat: {activeChatName || activeChat}</h3>
        <div class="chat-messages">
            {#if messages && messages.length > 0}
                {#each messages as message, i (i)}
                    {#if message.role !== 'function'}
                         <MessageBouble message={message.content} role={message.role} requestTime={`${parseFloat(respTimes[i]).toFixed(1)}s`} />
                    {/if}
                {/each}
            {/if}
        </div>
        <div class="input-box">
            <input bind:value={userInput}>
            <button on:click={sendMessage} class="send-btn">Send</button>
        </div>
        <div class={`no-conversation-overlay`} style="display: {conversations.length > 0 ? 'none' : 'flex'}">
            <button class="new-chat-btn" on:click={() => showModal = true}>New chat</button>
        </div>
    </div>


    <dialog
            bind:this={dialog}
            on:close={() => (showModal = false)}
            on:click|self={() => dialog.close()}
    >
        <div on:click|stopPropagation>
            <h3>Create new chat</h3>
            <hr />
            <div class="dialog-container">
                <h5>Chat name</h5>
                <input bind:value={chatName} />
            </div>
            <div class="dialog-container">
                <h5>Agent model</h5>
                <button class={`model-btn ${selectedModel === 'openai' ? 'active' : ''}`} on:click={() => selectedModel = "openai"}>OpenAI</button>
                <button class={`model-btn ${selectedModel === 'langchain' ? 'active' : ''}`} on:click={() => selectedModel = "langchain"}>Langchain</button>
            </div>
            <hr />
            <button autofocus on:click={() => dialog.close()}>Close modal</button>
            <button autofocus on:click={createNewConversation}>Create modal</button>
        </div>
    </dialog>

</div>

<style lang="scss">
  .content-container {
    display: flex;
    height: 100vh;
    overflow-y: hidden;
  }
    .conversation_history {
      width: 20vw;
      height: 100%;
      display: flex;
      background-color: rgb(42, 45, 48);
      padding: 1rem;
      padding-right: 0;
      flex-direction: column;

      h3 {
        color: lightgray;
      }

      ul {
        padding: 0;
        padding-top: 1rem;
        list-style: none;
        margin: 0;
        color: lightgray;
        cursor: pointer;
        overflow-y: auto;
        padding-bottom: 2rem;


        li {
          padding-top: 0.5rem;
          padding-bottom: 0.5rem;
          font-size: 0.9rem;
          padding-right: 0.8rem;

          span {
            font-weight: 600;
          }
        }
      }
    }

    .conversation {
      width: 80vw;
      display: flex;
      flex-direction: column;

      h1 {
        margin-left: 2rem;
        margin-bottom: 0;
      }

      h3 {
        margin-top: 0;
        margin-left: 2rem;
      }
    }

    .chat-messages {
      display: flex;
      flex-direction: column;
      overflow-y: auto;
      padding-right: 2rem;
      padding-left: 2rem;
      margin-bottom: 4.5rem;
    }

    .input-box {
      padding: 1rem;
      //height: 8rem;
      background-color:  rgba(42, 45, 48, 0.9);
      display: flex;
      flex-direction: row;
      align-content: center;
      gap: 1rem;
      position: fixed;
      bottom: 0;
      width: calc(100% - 20vw - 2.5rem);

      input {
        font-size: 1rem;
        border-radius: 0.5rem;
        border: none;
        outline: none;
        padding: 0.5rem;
        width: 100%;
      }

      .send-btn {
        background-color: rgb(78, 173, 69);
        color: white;
        border: none;
        outline: none;
        padding: 0.55rem 1rem 0.55rem 1rem;
        border-radius: 0.5rem;
        font-weight: bold;
        font-size: 1rem;

        &:hover {
          background-color: rgb(84, 187, 75);
        }

        &:active {
          background-color: rgb(110, 245, 97);
        }
      }
    }

  .new-chat-btn {
    font-size: 0.8rem;
    padding: 0.2rem;
    margin-bottom: 1rem;
    margin-right: 1rem;
  }

  .no-conversation-overlay {
    position: absolute;
    top: 5rem;
    background-color: rgba(0, 0, 0, 0.5);
    width: calc(100% - 20vw);
    height: calc(100% - 5rem);
    backdrop-filter: blur(0.4rem);
    display: flex;
    justify-content: center;
    align-content: center;
    align-items: center;
  }

  dialog {
    border: 1px solid darkgray;
    background-color: white;
    width: 30vw;
    border-radius: 0.2rem;

    h3 {
      margin: 0;
    }

    h5 {
      margin: 0;
      padding-bottom: 0.5rem;
    }
  }
  .model-btn {
    //color: white;
    border: none;
    outline: none;
    padding: 0.55rem 1rem 0.55rem 1rem;
    border-radius: 0.5rem;
    font-weight: bold;
    font-size: 1rem;
  }

  .dialog-container {
    margin-top: 1rem;
    margin-bottom: 1rem;
  }
  .active {
    background-color: rgb(78, 173, 69);
    color: white;
  }

  .active_chat {
    color: rgb(78, 173, 69);
  }
</style>