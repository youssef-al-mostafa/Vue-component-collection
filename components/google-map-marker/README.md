# Google Map Marker Component

A flexible and responsive Vue 3 component for embedding Google Maps with a single marker - without requiring an API key.

## Features

- 🌍 Embed Google Maps with a single marker in your Vue application
- 📍 Support for both coordinates (latitude/longitude) and Google Maps share links
- 🔑 No API key required - uses Google Maps embed API
- 📱 Fully responsive with customizable dimensions
- 🔍 Customizable zoom level
- ⚡ Optimized loading with lazy loading
- 🛡️ Error handling and validation
- 🔄 Reactive updates when props change

## Props

| Prop | Type | Default | Required | Description |
|------|------|---------|----------|-------------|
| `lat` | String | - | No* | Latitude coordinate |
| `long` | String | - | No* | Longitude coordinate |
| `link` | String | - | No* | Google Maps share URL (takes precedence over coordinates) |
| `width` | String | '100%' | No | Width of the map (with CSS units) |
| `height` | String | '400px' | No | Height of the map (with CSS units) |
| `zoom` | Number | 15 | No | Zoom level (1-20) |

*Either coordinates (`lat` AND `long`) OR a `link` must be provided.

## Usage

### Basic Usage with Coordinates

```vue
<template>
  <GoogleMapMarker
    lat="37.7749"
    long="-122.4194"
    height="500px"
  />
</template>

<script setup>
import GoogleMapMarker from '@/components/GoogleMapMarker/GoogleMapMarker.vue';
</script>
```

### Using with Google Maps Share Link

```vue
<template>
  <GoogleMapMarker
    link="https://www.google.com/maps/place/Eiffel+Tower/@48.8583701,2.2944813,17z/"
    width="100%"
    height="50vh"
    zoom="16"
  />
</template>

<script setup>
import GoogleMapMarker from '@/components/GoogleMapMarker/GoogleMapMarker.vue';
</script>
```

### Dynamic Example

```vue
<template>
  <GoogleMapMarker
    :lat="location.lat"
    :long="location.long"
    :height="mapHeight"
    :zoom="mapZoom"
  />
</template>

<script setup>
import { ref } from 'vue';
import GoogleMapMarker from '@/components/GoogleMapMarker/GoogleMapMarker.vue';

const location = ref({
  lat: '40.7128',
  long: '-74.0060'
});

const mapHeight = ref('400px');
const mapZoom = ref(12);
</script>
```

### Using in a Twig Template (Example)

```twig
{% if content.lat and content.long %}
  {{ 'GoogleMapMarker' | vue({
    lat: content.lat,
    long: content.long,
    height: '500px'
  }) }}
{% else %}
  {{ 'GoogleMapMarker' | vue({
    link: content.mapLink,
    height: '500px'
  }) }}
{% endif %}
```

## Implementation Details

- Uses Google's embed API instead of the JavaScript API to avoid authentication requirements
- The component prioritizes the `link` prop if both coordinates and link are provided

## Why Use This Component?

When you only need to display a single location marker on a map, using the Google Maps JavaScript API can be overkill and requires:
- Creating and managing an API key
- Setting up billing (even for free tier usage)
