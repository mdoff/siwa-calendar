<script lang="ts">
  import locationIcon from "./assets/location-icon.svg";

  let inputValue = "";
  let location: {
    latitude: number;
    longitude: number;
    address?: string;
  } | null = null;
  let locationLoading = false;
  let locationError = "";
  const timeZone = Intl.DateTimeFormat().resolvedOptions().timeZone;

  function handleSubmit() {
    if (inputValue.trim()) {
      console.log("Submitted:", inputValue);
      console.log("Location:", location);
      window.location.href = `https://calendar.google.com/calendar/u/0/r/eventedit?text=${inputValue}&dates=${new Date().toISOString()}&location=${location?.address}&ctz=${timeZone}`;
      inputValue = "";
    }
  }

  async function getLocation() {
    locationLoading = true;
    locationError = "";

    if (!navigator.geolocation) {
      locationError = "Geolocation is not supported by this browser.";
      locationLoading = false;
      return;
    }

    try {
      const position = await new Promise<GeolocationPosition>(
        (resolve, reject) => {
          navigator.geolocation.getCurrentPosition(resolve, reject, {
            enableHighAccuracy: true,
            timeout: 10000,
            maximumAge: 60000,
          });
        }
      );

      const { latitude, longitude } = position.coords;

      // Try to get address from coordinates using reverse geocoding
      try {
        const response = await fetch(
          `https://api.bigdatacloud.net/data/reverse-geocode-client?latitude=${latitude}&longitude=${longitude}&localityLanguage=en`
        );
        const data = await response.json();

        location = {
          latitude,
          longitude,
          address:
            data.display_name ||
            `${latitude.toFixed(6)}, ${longitude.toFixed(6)}`,
        };
      } catch (geocodeError) {
        // If reverse geocoding fails, just use coordinates
        location = {
          latitude,
          longitude,
          address: `${latitude.toFixed(6)}, ${longitude.toFixed(6)}`,
        };
      }
    } catch (error) {
      if (error instanceof GeolocationPositionError) {
        switch (error.code) {
          case error.PERMISSION_DENIED:
            locationError = "Location access denied by user.";
            break;
          case error.POSITION_UNAVAILABLE:
            locationError = "Location information is unavailable.";
            break;
          case error.TIMEOUT:
            locationError = "Location request timed out.";
            break;
          default:
            locationError = "An unknown error occurred while getting location.";
        }
      } else {
        locationError = "Failed to get location.";
      }
    } finally {
      locationLoading = false;
    }
  }
</script>

<header class="header">
  <div class="header-content">
    <h1 class="logo">Siwa Calendar</h1>
    <nav class="nav">
      <a href="/" class="nav-link">Home</a>
    </nav>
  </div>
</header>

<main class="main">
  <div class="container">
    <h2 class="title">Add New Event</h2>
    <form class="form" on:submit|preventDefault={handleSubmit}>
      <div class="input-group">
        <input
          type="text"
          class="input-field"
          placeholder="Enter event details..."
          bind:value={inputValue}
        />

        <div class="button-group">
          <button type="submit" class="submit-btn">
            <span class="btn-text">Add Event</span>
            <svg
              class="btn-icon"
              width="16"
              height="16"
              viewBox="0 0 16 16"
              fill="none"
            >
              <path
                d="M8 2V14M2 8H14"
                stroke="currentColor"
                stroke-width="2"
                stroke-linecap="round"
              />
            </svg>
          </button>

          <button
            type="button"
            class="location-btn"
            on:click={getLocation}
            disabled={locationLoading}
          >
            <span class="btn-text">
              {locationLoading ? "Getting Location..." : "Get Location"}
            </span>
            <img
              src={locationIcon}
              alt="Location Icon"
              class="btn-icon"
              width="16"
              height="16"
            />
          </button>
        </div>

        <!-- Location Display -->
        {#if location}
          <div class="location-display">
            <div class="location-icon">
              <img
                src={locationIcon}
                alt="Location Icon"
                width="18"
                height="18"
              />
            </div>
            <div class="location-info">
              <div class="location-address">{location.address}</div>
              <div class="location-coords">
                {location.latitude.toFixed(6)}, {location.longitude.toFixed(6)}
              </div>
            </div>
          </div>
        {/if}

        <!-- Location Error -->
        {#if locationError}
          <div class="error-message">
            <svg width="16" height="16" viewBox="0 0 16 16" fill="none">
              <circle
                cx="8"
                cy="8"
                r="7"
                stroke="currentColor"
                stroke-width="2"
              />
              <path
                d="M8 4v4M8 12h.01"
                stroke="currentColor"
                stroke-width="2"
                stroke-linecap="round"
              />
            </svg>
            {locationError}
          </div>
        {/if}
      </div>
    </form>
  </div>
</main>

<style>
  .header {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    padding: 1rem 0;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
    position: sticky;
    top: 0;
    z-index: 100;
  }

  .header-content {
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 1rem;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .logo {
    color: white;
    font-size: 1.8rem;
    font-weight: 600;
    margin: 0;
    text-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
  }

  .nav {
    display: flex;
    gap: 2rem;
  }

  .nav-link {
    color: rgba(255, 255, 255, 0.9);
    text-decoration: none;
    font-weight: 500;
    transition: color 0.3s ease;
    position: relative;
  }

  .nav-link:hover {
    color: white;
  }

  .nav-link::after {
    content: "";
    position: absolute;
    bottom: -4px;
    left: 0;
    width: 0;
    height: 2px;
    background: white;
    transition: width 0.3s ease;
  }

  .nav-link:hover::after {
    width: 100%;
  }

  .main {
    flex: 1;
    padding: 3rem 1rem;
    background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
    min-height: calc(100vh - 80px);
  }

  .container {
    margin: 0 auto;
    max-width: 600px;
  }

  .title {
    text-align: center;
    font-size: 2.5rem;
    font-weight: 700;
    color: #2d3748;
    margin-bottom: 2rem;
    text-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  }

  .form {
    background: white;
    padding: 2rem;
    border-radius: 16px;
    box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
    backdrop-filter: blur(10px);
  }

  .input-group {
    display: flex;
    flex-direction: column;
    gap: 1.5rem;
  }

  .button-group {
    display: flex;
    gap: 1.5rem;
  }

  .input-field {
    padding: 1rem 1.5rem;
    border: 2px solid #e2e8f0;
    border-radius: 12px;
    font-size: 1rem;
    background: #f8fafc;
    transition: all 0.3s ease;
    outline: none;
  }

  .input-field:focus {
    border-color: #667eea;
    background: white;
    box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
  }

  .input-field::placeholder {
    color: #a0aec0;
  }

  .submit-btn {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    border: none;
    padding: 1rem 2rem;
    border-radius: 12px;
    font-size: 1rem;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s ease;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 0.5rem;
    position: relative;
    overflow: hidden;
  }

  .submit-btn:hover {
    transform: translateY(-2px);
    box-shadow: 0 8px 25px rgba(102, 126, 234, 0.3);
  }

  .submit-btn:active {
    transform: translateY(0);
  }

  .submit-btn::before {
    content: "";
    position: absolute;
    top: 0;
    left: -100%;
    width: 100%;
    height: 100%;
    background: linear-gradient(
      90deg,
      transparent,
      rgba(255, 255, 255, 0.2),
      transparent
    );
    transition: left 0.5s;
  }

  .submit-btn:hover::before {
    left: 100%;
  }

  .btn-text {
    position: relative;
    z-index: 1;
  }

  .btn-icon {
    position: relative;
    z-index: 1;
    transition: transform 0.3s ease;
    filter: brightness(0) invert(1);
  }

  .submit-btn:hover .btn-icon {
    transform: rotate(90deg);
  }

  .location-btn {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    border: none;
    padding: 1rem 2rem;
    border-radius: 12px;
    font-size: 1rem;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s ease;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 0.5rem;
    position: relative;
    overflow: hidden;
  }

  .location-btn:hover {
    transform: translateY(-2px);
    box-shadow: 0 8px 25px rgba(102, 126, 234, 0.3);
  }

  .location-btn:active {
    transform: translateY(0);
  }

  .location-btn::before {
    content: "";
    position: absolute;
    top: 0;
    left: -100%;
    width: 100%;
    height: 100%;
    background: linear-gradient(
      90deg,
      transparent,
      rgba(255, 255, 255, 0.2),
      transparent
    );
    transition: left 0.5s;
  }

  .location-btn:hover::before {
    left: 100%;
  }

  .location-display {
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }

  .location-icon {
    width: 18px;
    height: 18px;
  }

  .location-icon img {
    width: 100%;
    height: 100%;
    filter: brightness(0) invert(1);
  }

  .location-info {
    display: flex;
    flex-direction: column;
  }

  .location-address {
    font-weight: 600;
  }

  .location-coords {
    color: #a0aec0;
  }

  .error-message {
    color: #e53e3e;
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }

  /* Mobile responsive design */
  @media (max-width: 768px) {
    .header-content {
      flex-direction: column;
      gap: 1rem;
      text-align: center;
    }

    .nav {
      gap: 1.5rem;
    }

    .logo {
      font-size: 1.5rem;
    }

    .title {
      font-size: 2rem;
    }

    .form {
      padding: 1.5rem;
      margin: 0 0.5rem;
    }

    .input-group {
      gap: 1.5rem;
    }

    .button-group {
      flex-direction: column;
      gap: 1rem;
    }

    .input-field {
      padding: 1.25rem 1rem;
      font-size: 1rem;
    }

    .submit-btn {
      padding: 1.25rem 2rem;
      font-size: 1.1rem;
    }

    .location-btn {
      padding: 1.25rem 2rem;
      font-size: 1.1rem;
    }
  }

  @media (max-width: 480px) {
    .header-content {
      padding: 0 0.5rem;
    }

    .nav {
      flex-wrap: wrap;
      gap: 1rem;
    }

    .main {
      padding: 2rem 0.5rem;
    }

    .form {
      padding: 1rem;
      margin: 0;
    }

    .title {
      font-size: 1.8rem;
      margin-bottom: 1.5rem;
    }

    .button-group {
      flex-direction: column;
      gap: 1rem;
    }

    .location-btn {
      padding: 1.25rem 2rem;
      font-size: 1.1rem;
    }
  }

  /* Dark mode support */
  @media (prefers-color-scheme: dark) {
    .main {
      background: linear-gradient(135deg, #1a202c 0%, #2d3748 100%);
    }

    .title {
      color: white;
    }

    .form {
      background: rgba(45, 55, 72, 0.8);
      backdrop-filter: blur(10px);
    }

    .input-field {
      background: #2d3748;
      border-color: #4a5568;
      color: white;
    }

    .input-field:focus {
      background: #1a202c;
      border-color: #667eea;
    }

    .input-field::placeholder {
      color: #718096;
    }

    .submit-btn {
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    }

    .location-btn {
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    }

    .location-display {
      color: white;
    }

    .location-address {
      font-weight: 600;
    }

    .location-coords {
      color: #a0aec0;
    }

    .error-message {
      color: #e53e3e;
    }
  }

  .location-btn:hover .btn-icon {
    transform: rotate(15deg) scale(1.1);
  }
</style>
