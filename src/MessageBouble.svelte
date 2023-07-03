<script lang="ts">
    export let message: string = ''
    export let role: string = ''
    export let requestTime: string = ''

</script>

<div class={`message-bubble message-bubble-${role}`}>
    <div class="header">
        <h4><span>Role:</span>{role}</h4>
        {#if requestTime && requestTime !== "Unknow"}
            <h4><span>Response time:</span> {requestTime}</h4>
        {/if}
    </div>
    <div class={`message ${role}`}>
        {#if message != null}
            <p>{message}</p>
        {:else }
            <p>Function call</p>
        {/if}
    </div>
</div>

<style lang="scss">
  .header {
    display: flex;
    gap: 1rem;
    color: gray;
    margin-bottom: 0.1rem;
    font-size: 0.9rem;
    margin-left: 0.5rem;

    h4 {
      font-weight: 400;
      margin: 0;
    }

    span {
      font-weight: 500;
      color: rgb(18, 19, 20);
      padding-right: 0.5rem;
    }
  }

 .message-bubble {
    width: 100%;
    margin-top: 0.5rem;
   margin-bottom: 0.5rem;
 }

    .message {
      border-radius: 0.5rem;
      padding: 0.8rem;
      display: flex;
      height: max-content;

      p {
        margin: 0;
      }
    }

    .system {
      background-color: rgba(217, 216, 212,0.5);
    }

  .ai {
    background-color: rgba(217, 216, 212,0.5);
  }

  .assistant {
    background-color: rgba(217, 216, 212,0.5);
  }

  .user {
      background-color: rgba(125, 178, 240,0.3);
    }

    .message-bubble-system {
      float: left;
    }

    .message-bubble-ai {
      float: left;
    }

  .message-bubble-user {
    float: right;

    .header {
      justify-content: flex-end;
    }

    .message {
      justify-content: flex-end;
    }


  }
</style>