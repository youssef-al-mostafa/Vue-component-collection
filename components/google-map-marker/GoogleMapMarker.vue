<script setup lang="ts">
import { ref, watch, onMounted } from 'vue';

interface GoogleMapMarkerProps {
  lat?: string;
  long?: string;
  link?: string;
  width?: string;
  height?: string;
  zoom?: number;
}

const props = withDefaults(defineProps<GoogleMapMarkerProps>(), {
  width: '100%',
  height: '400px',
  zoom: 15
});

const embedUrl = ref<string>('');
const isLoading = ref<boolean>(true);
const hasError = ref<boolean>(false);
const errorMessage = ref<string>('');

/**
 * Create a Google Maps embed URL from latitude and longitude
 */
const createMapUrl = (lat: string, long: string, zoom: number): string => {
  return `https://maps.google.com/maps?q=${lat},${long}&z=${zoom}&output=embed`;
};

/**
 * Extract embed URL from a Google Maps share link
 * or return null if it's not a valid Google Maps link
 */
const extractEmbedUrl = (shareLink: string): string | null => {
  if (shareLink.includes('google.com/maps')) {
    if (shareLink.includes('maps/place/')) {
      return shareLink.replace(/\/maps\/place\//, '/maps/embed/v1/place?key=');
    } else if (shareLink.includes('maps/search/')) {
      return shareLink.replace(/\/maps\/search\//, '/maps/embed/v1/search?key=');
    } else if (shareLink.includes('@')) {
      try {
        const coordMatch = shareLink.match(/@(-?\d+\.\d+),(-?\d+\.\d+)/);
        if (coordMatch) {
          const [, lat, lng] = coordMatch;
          return createMapUrl(lat, lng, props.zoom);
        }
      } catch (error) {
      }
    }
    
    return shareLink.includes('output=embed') ? 
      shareLink : 
      `${shareLink.replace(/\?(.*)/, '')}?output=embed`;
  }
  
  return null;
};

/**
 * Update the map URL based on props
 */
const updateMapUrl = (): void => {
  isLoading.value = true;
  hasError.value = false;
  
  try {
    if (props.link) {
      const embeddedUrl = extractEmbedUrl(props.link);
      if (embeddedUrl) {
        embedUrl.value = embeddedUrl;
        return;
      } else {
        console.warn('Invalid Google Maps link provided, falling back to coordinates if available');
      }
    }
    
    if (props.lat && props.long) {
      embedUrl.value = createMapUrl(props.lat, props.long, props.zoom);
      return;
    }
    
    hasError.value = true;
    errorMessage.value = 'Please provide either valid coordinates (lat/long) or a Google Maps link';
    embedUrl.value = '';
  } catch (error) {
    hasError.value = true;
    errorMessage.value = 'Error processing map data';
    console.error('Error in GoogleMap component:', error);
  }
};

watch(
  () => [props.lat, props.long, props.link, props.zoom],
  () => updateMapUrl(),
  { immediate: true }
);

const onIframeLoad = () => {
  isLoading.value = false;
};

onMounted(() => {
  updateMapUrl();
});
</script>

<template>
  <div class="google-map" :class="{ 'google-map--error': hasError }">
    <div v-if="isLoading" class="google-map__loading">
      <div class="google-map__loading-spinner"></div>
      <span>Loading map...</span>
    </div>
    
    <div v-if="hasError" class="google-map__error">
      <p>{{ errorMessage }}</p>
    </div>
    
    <iframe
      v-if="embedUrl && !hasError"
      :src="embedUrl"
      :width="width"
      :height="height"
      class="google-map__iframe"
      frameborder="0"
      allowfullscreen="true"
      referrerpolicy="no-referrer-when-downgrade"
      loading="lazy"
      @load="onIframeLoad"
    ></iframe>
  </div>
</template>

<style lang="scss">
.google-map {
  position: relative;
  width: 100%;
  
  &__iframe {
    border: 0;
    display: block;
  }
  
  &__loading {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    background-color: rgba(240, 240, 240, 0.7);
    z-index: 1;
    
    &-spinner {
      width: 40px;
      height: 40px;
      border: 4px solid #f3f3f3;
      border-top: 4px solid #3498db;
      border-radius: 50%;
      animation: google-map-spin 1s linear infinite;
      margin-bottom: 10px;
    }
  }
  
  &__error {
    padding: 20px;
    background-color: #ffebee;
    color: #d32f2f;
    border: 1px solid #ffcdd2;
    border-radius: 4px;
    margin: 10px 0;
  }
  
  @keyframes google-map-spin {
    0% { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
  }
}
</style>