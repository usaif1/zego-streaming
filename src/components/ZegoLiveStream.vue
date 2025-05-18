<script setup lang="ts">
import { onMounted, ref, computed } from 'vue';
import { ZegoUIKitPrebuilt } from '@zegocloud/zego-uikit-prebuilt';

const rootElement = ref<HTMLElement | null>(null);
const role = ref(ZegoUIKitPrebuilt.Host); // Use static enum property
const roomID = ref('fuelbuddyRoom');
const showCopiedMessage = ref(false);

// ZEGO LiveStream Configuration
const appID = 1829346243;
const serverSecret = '26d48a0fa1538ef1e823cef951ef0e69';

const shareableLink = computed(() => {
  const url = new URL(window.location.href);
  url.searchParams.set('room', roomID.value);
  url.searchParams.set('role', 'Audience');
  return url.toString();
});

const copyLinkToClipboard = () => {
  navigator.clipboard.writeText(shareableLink.value).then(() => {
    showCopiedMessage.value = true;
    setTimeout(() => {
      showCopiedMessage.value = false;
    }, 2000);
  });
};

const toggleRole = () => {
  role.value = role.value === ZegoUIKitPrebuilt.Host ? ZegoUIKitPrebuilt.Audience : ZegoUIKitPrebuilt.Host;
  joinRoom();
};

const joinRoom = () => {
  if (!rootElement.value) return;
  
  // Check URL parameters for room and role
  const urlParams = new URLSearchParams(window.location.search);
  const roomParam = urlParams.get('room');
  const roleParam = urlParams.get('role');
  
  if (roomParam) {
    roomID.value = roomParam;
  }
  
  if (roleParam === 'Host') {
    role.value = ZegoUIKitPrebuilt.Host;
  } else if (roleParam === 'Audience') {
    role.value = ZegoUIKitPrebuilt.Audience;
  }
  
  const userID = String(Math.floor(Math.random() * 10000));
  const userName = `user_${userID}`;

  // Get token
  const kitToken = ZegoUIKitPrebuilt.generateKitTokenForTest(
    appID,
    serverSecret,
    roomID.value,
    userID,
    userName
  );

  // Create instance
  const zp = ZegoUIKitPrebuilt.create(kitToken);

  // Clear previous instance if exists
  rootElement.value.innerHTML = '';

  // Join room with appropriate role
  zp.joinRoom({
    container: rootElement.value,
    turnOnCameraWhenJoining: role.value === ZegoUIKitPrebuilt.Host,
    showMyCameraToggleButton: role.value === ZegoUIKitPrebuilt.Host,
    showAudioVideoSettingsButton: role.value === ZegoUIKitPrebuilt.Host,
    showScreenSharingButton: false,
    showTextChat: false,
    showUserList: false,
    scenario: {
      mode: ZegoUIKitPrebuilt.LiveStreaming,
      config: {
        role: role.value,
      },
    },
  });
};

onMounted(() => {
  joinRoom();
});
</script>

<template>
  <div>
    <div class="role-selector">
      <h1>FuelBuddy Live Streaming - {{ role === ZegoUIKitPrebuilt.Host ? 'Host' : 'Viewer' }} Mode</h1>
      <div class="controls">
        <button @click="toggleRole" class="role-toggle">
          Switch to {{ role === ZegoUIKitPrebuilt.Host ? 'Viewer' : 'Host' }} Mode
        </button>
        
        <div v-if="role === ZegoUIKitPrebuilt.Host" class="share-section">
          <input type="text" readonly :value="shareableLink" class="share-link" />
          <button @click="copyLinkToClipboard" class="copy-button">
            {{ showCopiedMessage ? 'Copied!' : 'Copy Viewer Link' }}
          </button>
        </div>
      </div>
    </div>
    <div ref="rootElement" class="live-container"></div>
  </div>
</template>

<style scoped>
.role-selector {
  margin-bottom: 1rem;
}

.controls {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: 1rem 0;
  flex-wrap: wrap;
  gap: 1rem;
}

.role-toggle {
  padding: 0.5rem 1rem;
  background-color: #3498db;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1rem;
}

.role-toggle:hover {
  background-color: #2980b9;
}

.share-section {
  display: flex;
  gap: 0.5rem;
  flex: 1;
  max-width: 600px;
}

.share-link {
  flex: 1;
  padding: 0.5rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 0.9rem;
}

.copy-button {
  padding: 0.5rem 1rem;
  background-color: #27ae60;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  white-space: nowrap;
}

.copy-button:hover {
  background-color: #2ecc71;
}

.live-container {
  width: 100%;
  height: 600px;
  background-color: #f2f2f2;
  border-radius: 8px;
  overflow: hidden;
}
</style> 