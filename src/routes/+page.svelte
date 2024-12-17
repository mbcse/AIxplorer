<script lang="ts">
  import { useChat } from '@ai-sdk/svelte';

  const {
    error,
    input,
    isLoading,
    handleSubmit,
    messages,
    reload,
    stop
  } = useChat({
    onFinish(message, { usage, finishReason }) {
      console.log('Message from agent:', message);
      console.log('Usage', usage);
      console.log('FinishReason', finishReason);
    },
  });

  function formatAssistantMessage(content: string) {
    const sections = content.split('\n\n');
    let formattedContent = '';
    
    sections.forEach(section => {
      if (section.includes('TRANSACTION OVERVIEW:')) {
        formattedContent += `<div class="section overview">
          <h3 class="section-title">
            <span class="icon">🔍</span>
            <span>Transaction Overview</span>
            ${extractComplexityBadge(section)}
          </h3>
          ${formatList(section.replace('TRANSACTION OVERVIEW:', '').trim())}
        </div>`;
      }
      else if (section.includes('NETWORK DETAILS:')) {
        formattedContent += `<div class="section network">
          <h3 class="section-title">
            <span class="icon">🌐</span>
            <span>Network Details</span>
          </h3>
          ${formatList(section.replace('NETWORK DETAILS:', '').trim())}
        </div>`;
      }
      else if (section.includes('TRANSFER ANALYSIS:')) {
        formattedContent += `<div class="section transfers">
          <h3 class="section-title">
            <span class="icon">↔️</span>
            <span>Transfer Analysis</span>
          </h3>
          ${formatTransferSection(section.replace('TRANSFER ANALYSIS:', '').trim())}
        </div>`;
      }
      else if (section.includes('DEX INTERACTIONS:')) {
        formattedContent += `<div class="section dex">
          <h3 class="section-title">
            <span class="icon">🔄</span>
            <span>DEX Interactions</span>
          </h3>
          ${formatList(section.replace('DEX INTERACTIONS:', '').trim())}
        </div>`;
      }
      else if (section.includes('CONTRACT INTERACTIONS:')) {
        formattedContent += `<div class="section contract">
          <h3 class="section-title">
            <span class="icon">📝</span>
            <span>Contract Interactions</span>
          </h3>
          ${formatList(section.replace('CONTRACT INTERACTIONS:', '').trim())}
        </div>`;
      }
      else if (section.includes('COST ANALYSIS:')) {
        formattedContent += `<div class="section cost">
          <h3 class="section-title">
            <span class="icon">⛽</span>
            <span>Cost Analysis</span>
          </h3>
          ${formatList(section.replace('COST ANALYSIS:', '').trim())}
        </div>`;
      }
      else if (section.includes('SECURITY ASSESSMENT:')) {
        formattedContent += `<div class="section security">
          <h3 class="section-title">
            <span class="icon">🛡️</span>
            <span>Security Assessment</span>
            ${extractRiskBadge(section)}
          </h3>
          ${formatList(section.replace('SECURITY ASSESSMENT:', '').trim())}
        </div>`;
      }
      else if (section.includes('ADDITIONAL INSIGHTS:')) {
        formattedContent += `<div class="section insights">
          <h3 class="section-title">
            <span class="icon">💡</span>
            <span>Additional Insights</span>
          </h3>
          ${formatList(section.replace('ADDITIONAL INSIGHTS:', '').trim())}
        </div>`;
      }
    });
    
    return formattedContent;
  }

  function extractComplexityBadge(content: string): string {
    const complexityMatch = content.match(/complexity score: (Simple|Moderate|Complex|Very Complex)/i);
    if (!complexityMatch) return '';
    
    const complexity = complexityMatch[1].toLowerCase();
    const colorClass = {
      'simple': 'bg-green-100 text-green-800',
      'moderate': 'bg-blue-100 text-blue-800',
      'complex': 'bg-yellow-100 text-yellow-800',
      'very complex': 'bg-red-100 text-red-800'
    }[complexity] || 'bg-gray-100 text-gray-800';

    return `<span class="complexity-badge ${colorClass}">${complexityMatch[1]}</span>`;
  }

  function extractRiskBadge(content: string): string {
    const riskMatch = content.match(/risk level: (Low|Medium|High)/i);
    if (!riskMatch) return '';
    
    const risk = riskMatch[1].toLowerCase();
    const colorClass = {
      'low': 'bg-green-100 text-green-800',
      'medium': 'bg-yellow-100 text-yellow-800',
      'high': 'bg-red-100 text-red-800'
    }[risk] || 'bg-gray-100 text-gray-800';

    return `<span class="risk-badge ${colorClass}">${riskMatch[1]} Risk</span>`;
  }

  function formatTransferSection(content: string) {
    const parts = content.split('\n\n');
    let html = '';
    
    parts.forEach(part => {
      if (part.includes('Native Currency:')) {
        html += `<div class="transfer-group native">
          <h4 class="transfer-title">
            <span class="icon">💰</span>
            <span>Native Currency Transfer</span>
          </h4>
          ${formatList(part.replace('Native Currency:', '').trim())}
        </div>`;
      }
      else if (part.includes('Token Transfers (ERC20):')) {
        html += `<div class="transfer-group erc20">
          <h4 class="transfer-title">
            <span class="icon">🪙</span>
            <span>Token Transfers</span>
          </h4>
          ${formatList(part.replace('Token Transfers (ERC20):', '').trim())}
        </div>`;
      }
      else if (part.includes('NFT Transfers')) {
        html += `<div class="transfer-group nft">
          <h4 class="transfer-title">
            <span class="icon">🖼️</span>
            <span>NFT Transfers</span>
          </h4>
          ${formatList(part.replace('NFT Transfers (ERC721/ERC1155):', '').trim())}
        </div>`;
      }
    });
    
    return html || '<div class="no-transfers">No transfers detected in this transaction</div>';
  }

  function formatList(content: string) {
    return content.split('\n')
      .map(item => item.trim())
      .filter(item => item.length > 0)
      .map(item => {
        const formattedItem = item.replace(/^-\s*/, '');
        if (formattedItem.toLowerCase().includes('warning') || formattedItem.toLowerCase().includes('error')) {
          return `<div class="list-item warning">${formattedItem}</div>`;
        } else if (formattedItem.toLowerCase().includes('success')) {
          return `<div class="list-item success">${formattedItem}</div>`;
        }
        return `<div class="list-item">${formattedItem}</div>`;
      })
      .join('');
  }

  function formatAddress(address: string): string {
    return `${address.slice(0, 6)}...${address.slice(-4)}`;
  }
</script>

<svelte:head>
  <title>AIxplorer | Web3 Transaction Analyzer</title>
  <meta name="description" content="AI-Powered Blockchain Transaction Analysis" />
</svelte:head>

<section>
  <div class="header">
    <h1>AIxplorer</h1>
    <p class="subtitle">AI-Powered Blockchain Transaction Analysis</p>
  </div>
  
  <div class="chat-container">
    <div class="messages-container">
      {#each $messages as message}
        <div class="message {message.role}">
          <div class="avatar {message.role}">
            {message.role === 'assistant' ? '🤖' : '👤'}
          </div>
          <div class="content">
            <div class="sender">{message.role === 'assistant' ? 'AIxplorer AI' : 'You'}</div>
            {#if message.role === 'assistant'}
              {@html formatAssistantMessage(message.content)}
            {:else}
              <div class="message-text">{message.content}</div>
            {/if}
          </div>
        </div>
      {/each}
      
      {#if $messages.length === 0}
        <div class="welcome-message">
          <h2>Welcome to AIxplorer!</h2>
          <p>Enter a transaction hash and chain ID to analyze blockchain transactions.</p>
          <div class="example">
            <strong>Example:</strong>
            <code>analyze 0x123... on chain 1</code>
          </div>
        </div>
      {/if}
    </div>

    {#if $isLoading}
      <div class="loading">
        <div class="spinner"></div>
        <div class="loading-text">Analyzing blockchain transaction...</div>
        <button type="button" class="stop-button" on:click={stop}>
          Stop Analysis
        </button>
      </div>
    {/if}

    {#if $error}
      <div class="error">
        <div class="error-icon">⚠️</div>
        <div class="error-text">Analysis failed. Please try again.</div>
        <button type="button" class="retry-button" on:click={reload}>
          Retry Analysis
        </button>
      </div>
    {/if}

    <form on:submit={handleSubmit} class="input-form">
      <input
        bind:value={$input}
        placeholder="Enter transaction hash and chain ID (e.g., analyze 0x123... on chain 1)"
        disabled={$isLoading || $error != null}
        class="main-input"
      />
      <button 
        type="submit" 
        disabled={$isLoading || $error != null}
        class="submit-button"
      >
        {$isLoading ? 'Analyzing...' : 'Analyze Transaction'}
      </button>
    </form>
  </div>
</section>

<style>
  :global(*) {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }

  section {
    display: flex;
    flex-direction: column;
    align-items: center;
    min-height: 100vh;
    background-color: #f8fafc;
    padding: 2rem 1rem;
    font-family: system-ui, -apple-system, sans-serif;
  }

  .header {
    text-align: center;
    margin-bottom: 2rem;
  }

  h1 {
    font-size: 2.5rem;
    color: #1a237e;
    margin-bottom: 0.5rem;
    font-weight: 700;
  }

  .subtitle {
    color: #64748b;
    font-size: 1.1rem;
  }

  .chat-container {
    width: 100%;
    max-width: 1000px;
    background: white;
    border-radius: 1rem;
    box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1);
    overflow: hidden;
  }

  .messages-container {
    height: 70vh;
    overflow-y: auto;
    padding: 2rem;
    scroll-behavior: smooth;
  }

  .welcome-message {
    text-align: center;
    padding: 2rem;
    background: #f8fafc;
    border-radius: 0.5rem;
    margin: 2rem 0;
  }

  .welcome-message h2 {
    color: #1e293b;
    margin-bottom: 1rem;
  }

  .welcome-message .example {
    margin-top: 1rem;
    padding: 1rem;
    background: #e2e8f0;
    border-radius: 0.5rem;
    display: inline-block;
  }

  .welcome-message code {
    background: #cbd5e1;
    padding: 0.25rem 0.5rem;
    border-radius: 0.25rem;
    margin-left: 0.5rem;
  }

  .message {
    display: flex;
    gap: 1rem;
    margin-bottom: 1.5rem;
    animation: fadeIn 0.3s ease-out;
  }

  .avatar {
    width: 40px;
    height: 40px;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 1.5rem;
    flex-shrink: 0;
  }

  .avatar.assistant {
    background: #e8f0fe;
  }

  .avatar.user {
    background: #f0fdf4;
  }

  .content {
    flex: 1;
    max-width: calc(100% - 60px);
  }

  .sender {
    font-weight: 500;
    color: #1e293b;
    margin-bottom: 0.5rem;
  }

  .section {
    background: #ffffff;
    border: 1px solid #e2e8f0;
    border-radius: 0.5rem;
    padding: 1rem;
    margin-bottom: 1rem;
  }

  .section-title {
    color: #1e293b;
    font-size: 1.1rem;
    margin-bottom: 0.75rem;
    display: flex;
    align-items: center;
    gap: 0.5rem;
    font-weight: 600;
  }

  .icon {
    font-size: 1.2rem;
    line-height: 1;
  }

  .complexity-badge,
  .risk-badge {
    margin-left: auto;
    padding: 0.25rem 0.75rem;
    border-radius: 1rem;
    font-size: 0.875rem;
    font-weight: 500;
  }

  .list-item {
    margin: 0.5rem 0;
    line-height: 1.5;
    color: #475569;
    padding: 0.5rem;
    border-radius: 0.25rem;
  }

  .list-item.warning {
    background: #fff5f5;
    color: #c53030;
    padding: 0.75rem;
    border-left: 4px solid #fc8181;
  }

  .list-item.success {
    background: #f0fff4;
    color: #2f855a;
    padding: 0.75rem;
    border-left: 4px solid #68d391;

  }

.transfer-group {
  margin-bottom: 1rem;
  padding: 1rem;
  border-radius: 0.5rem;
}

.transfer-title {
  font-size: 1rem;
  color: #1e293b;
  margin-bottom: 0.75rem;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-weight: 500;
}

.transfer-group.native {
  background: #f0f9ff;
  border-left: 4px solid #3b82f6;
}

.transfer-group.erc20 {
  background: #f0fdf4;
  border-left: 4px solid #10b981;
}

.transfer-group.nft {
  background: #fdf2f8;
  border-left: 4px solid #ec4899;
}

.no-transfers {
  padding: 1rem;
  color: #64748b;
  text-align: center;
  background: #f8fafc;
  border-radius: 0.5rem;
  font-style: italic;
}

.input-form {
  display: flex;
  gap: 1rem;
  padding: 1.5rem;
  background: #ffffff;
  border-top: 1px solid #e2e8f0;
  position: relative;
}

.main-input {
  flex: 1;
  padding: 1rem 1.5rem;
  border: 1px solid #e2e8f0;
  border-radius: 0.75rem;
  font-size: 1rem;
  transition: all 0.2s;
  background: #ffffff;
  color: #1e293b;
}

.main-input:focus {
  outline: none;
  border-color: #3b82f6;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
}

.main-input:disabled {
  background: #f8fafc;
  cursor: not-allowed;
}

.submit-button {
  padding: 1rem 2rem;
  background: #3b82f6;
  color: white;
  border: none;
  border-radius: 0.75rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
  white-space: nowrap;
}

.submit-button:hover:not(:disabled) {
  background: #2563eb;
  transform: translateY(-1px);
}

.submit-button:active:not(:disabled) {
  transform: translateY(0);
}

.submit-button:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

.loading {
  text-align: center;
  padding: 1.5rem;
  background: #f8fafc;
  border-radius: 0.5rem;
  margin: 1rem;
  animation: fadeIn 0.3s ease-out;
}

.loading-text {
  color: #1e293b;
  margin: 1rem 0;
  font-weight: 500;
}

.spinner {
  width: 2.5rem;
  height: 2.5rem;
  border: 3px solid #e2e8f0;
  border-top-color: #3b82f6;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin: 0 auto;
}

.stop-button {
  background: #ef4444;
  color: white;
  border: none;
  padding: 0.75rem 1.5rem;
  border-radius: 0.5rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
  margin-top: 1rem;
}

.stop-button:hover {
  background: #dc2626;
}

.error {
  text-align: center;
  padding: 1.5rem;
  background: #fef2f2;
  border-radius: 0.5rem;
  margin: 1rem;
  animation: fadeIn 0.3s ease-out;
}

.error-icon {
  font-size: 2rem;
  margin-bottom: 0.5rem;
}

.error-text {
  color: #991b1b;
  margin-bottom: 1rem;
  font-weight: 500;
}

.retry-button {
  background: #10b981;
  color: white;
  border: none;
  padding: 0.75rem 1.5rem;
  border-radius: 0.5rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
}

.retry-button:hover {
  background: #059669;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

@media (max-width: 768px) {
  .chat-container {
    border-radius: 0;
  }
  
  .input-form {
    flex-direction: column;
  }
  
  .submit-button {
    width: 100%;
  }

  .section {
    padding: 0.75rem;
  }

  .complexity-badge,
  .risk-badge {
    font-size: 0.75rem;
  }

  .messages-container {
    height: calc(100vh - 200px);
    padding: 1rem;
  }
}

@media (max-width: 480px) {
  h1 {
    font-size: 2rem;
  }

  .subtitle {
    font-size: 1rem;
  }

  .section-title {
    font-size: 1rem;
  }

  .main-input {
    font-size: 0.875rem;
    padding: 0.75rem 1rem;
  }
}

@media (prefers-reduced-motion: reduce) {
  .spinner {
    animation: none;
  }

  .message {
    animation: none;
  }

  .loading,
  .error {
    animation: none;
  }
}
</style>