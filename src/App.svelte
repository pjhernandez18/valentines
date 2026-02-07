<script lang="ts">
  import { onMount } from 'svelte';

  // States
  type GameState = 'login' | 'loading' | 'asking' | 'wrong-answer' | 'second-meme' | 'try-again' | 'success-gif1' | 'success-gif2' | 'success';
  let state: GameState = 'login';

  // Fake loading GIF - the happy dance
  const LOADING_IMAGE = '/gifs/loading-dance.gif';
  let hearts: { id: number; x: number; delay: number; duration: number; size: number }[] = [];
  let celebrationHearts: { id: number; x: number; y: number; rotation: number }[] = [];

  // Login state
  let password = '';
  let loginError = '';
  let isLoading = false;
  const CORRECT_PASSWORD = 'bitchinbayonne';
  // No button state
  let noHoverCount = 0;
  let noButtonPosition = { x: 0, y: 0 };
  let noButtonMoved = false;

  // Slideshow images for the main page
  const SLIDESHOW_IMAGES = [
    { url: '/gifs/selena-gomez.gif', text: 'u better say yes' },
    { url: '/gifs/jalen-brunson.gif', text: 'lemme shoot my shot mami' },
    { url: '/gifs/eternal-sunshine.gif', text: 'Meet me in Montauk' },
    { url: '/gifs/brat.gif', text: '' },
  ];
  let currentSlide = 0;
  let slideInterval: ReturnType<typeof setInterval>;

  // Meme URLs for the No button sequence
  const WRONG_ANSWER_GIF = '/gifs/stitch-rain.gif';
  const SECOND_MEME_GIF = '/gifs/pride-n-prejudice.gif';
  const TRY_AGAIN_GIF = '/gifs/try-again.gif';

  // Success sequence
  const SUCCESS_GIF_1 = '/gifs/bad-bunny.gif';
  const SUCCESS_GIF_2 = '/gifs/paul-revere.gif';
  const FINAL_PHOTO = '/images/final-photo.png';

  // Tiredness levels - every 3 hovers it gets more tired
  $: tirednessLevel = Math.floor(noHoverCount / 3);

  // Button text based on tiredness
  $: noButtonText = (() => {
    if (tirednessLevel === 0) return 'No';
    if (tirednessLevel === 1) return 'Getting tired... 😓';
    if (tirednessLevel === 2) return 'So tired... 😩';
    if (tirednessLevel === 3) return '*wheeze* 😵';
    if (tirednessLevel === 4) return 'Exhausted... 😵‍💫';
    return "I give up... 💀";
  })();

  // Check if button has given up (stationary)
  $: hasGivenUp = tirednessLevel >= 5;

  // Transition speed: instant at first, then progressively slower
  $: transitionSpeed = (() => {
    if (noHoverCount <= 1) return 0; // INSTANT - impossible to catch!
    if (tirednessLevel === 1) return 0.1; // Still fast
    if (tirednessLevel === 2) return 0.25; // Slowing
    if (tirednessLevel === 3) return 0.5; // Tired
    if (tirednessLevel === 4) return 0.8; // Exhausted
    return 0; // Given up - no movement
  })();

  function createHearts() {
    hearts = Array.from({ length: 20 }, (_, i) => ({
      id: i,
      x: Math.random() * 100,
      delay: Math.random() * 5,
      duration: 3 + Math.random() * 4,
      size: 10 + Math.random() * 20,
    }));
  }

  function startSlideshow() {
    currentSlide = 0;
    slideInterval = setInterval(() => {
      currentSlide = (currentSlide + 1) % SLIDESHOW_IMAGES.length;
    }, 3000); // Change slide every 3 seconds
  }

  function stopSlideshow() {
    if (slideInterval) {
      clearInterval(slideInterval);
    }
  }

  function resetInterval() {
    // Reset the timer when manually changing slides
    stopSlideshow();
    slideInterval = setInterval(() => {
      currentSlide = (currentSlide + 1) % SLIDESHOW_IMAGES.length;
    }, 3000);
  }

  function nextSlide() {
    currentSlide = (currentSlide + 1) % SLIDESHOW_IMAGES.length;
    resetInterval();
  }

  function prevSlide() {
    currentSlide = (currentSlide - 1 + SLIDESHOW_IMAGES.length) % SLIDESHOW_IMAGES.length;
    resetInterval();
  }

  function goToSlide(i: number) {
    currentSlide = i;
    resetInterval();
  }

  function handleLogin() {
    loginError = '';
    isLoading = true;

    // Fake loading to make it seem like a real system
    setTimeout(() => {
      if (password.toLowerCase() === CORRECT_PASSWORD.toLowerCase()) {
        // Show the troll loading screen first
        state = 'loading';
        isLoading = false;

        // After 3 seconds, reveal the Valentine's app
        setTimeout(() => {
          state = 'asking';
          createHearts();
          startSlideshow();
        }, 3000);
      } else {
        loginError = 'Invalid credentials. Please contact IT support.';
        isLoading = false;
      }
    }, 1500);
  }

  function handleKeydown(event: KeyboardEvent) {
    if (event.key === 'Enter' && state === 'login') {
      handleLogin();
    }
  }

  function moveNoButton() {
    // Don't move if given up
    if (hasGivenUp) return;

    noHoverCount++;
    noButtonMoved = true;

    // Random position ANYWHERE on screen (with padding for button size)
    const padding = 120;
    const maxX = window.innerWidth - padding;
    const maxY = window.innerHeight - padding;

    // Generate completely random position across entire screen
    let newX = padding + Math.random() * (maxX - padding);
    let newY = padding + Math.random() * (maxY - padding);

    noButtonPosition = { x: newX, y: newY };
  }

  function handleNoClick() {
    // Stop slideshow during meme sequence
    stopSlideshow();

    // Show wrong answer sequence
    // Step 1: Stitch in rain for 5 seconds
    state = 'wrong-answer';

    // Step 2: Fumble GIF for 5 seconds
    setTimeout(() => {
      state = 'second-meme';
    }, 5000);

    // Step 3: Back to asking page with "try again" overlay for 3 seconds
    setTimeout(() => {
      state = 'try-again';
      startSlideshow();
    }, 10000);

    // Step 4: Back to normal
    setTimeout(() => {
      state = 'asking';
      // Don't reset hover count - button stays tired!
    }, 13000);
  }

  function handleYesClick() {
    stopSlideshow();

    // Step 1: First GIF (dog) - fade in, 3 seconds
    state = 'success-gif1';

    // Step 2: Second GIF (Dwight) - 3 seconds
    setTimeout(() => {
      state = 'success-gif2';
    }, 3000);

    // Step 3: Final photo with celebration
    setTimeout(() => {
      state = 'success';
      // Create celebration hearts explosion
      celebrationHearts = Array.from({ length: 50 }, (_, i) => ({
        id: i,
        x: Math.random() * 100,
        y: Math.random() * 100,
        rotation: Math.random() * 360,
      }));
    }, 6000);
  }
</script>

<svelte:window on:keydown={handleKeydown} />

<main class:login-page={state === 'login'} class:loading-page={state === 'loading'}>
  {#if state === 'login'}
    <!-- DISGUISED LOGIN PAGE - Looks like boring IT stuff -->
    <div class="login-container">
      <div class="login-header">
        <div class="company-logo">
          <svg viewBox="0 0 24 24" width="48" height="48" fill="#1a73e8">
            <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-2 15l-5-5 1.41-1.41L10 14.17l7.59-7.59L19 8l-9 9z"/>
          </svg>
        </div>
        <h1 class="login-title">System Verification Required</h1>
        <p class="login-subtitle">IT Security Compliance Portal v2.4.1</p>
      </div>

      <div class="login-card">
        <div class="security-badge">
          <svg viewBox="0 0 24 24" width="20" height="20" fill="#5f6368">
            <path d="M12 1L3 5v6c0 5.55 3.84 10.74 9 12 5.16-1.26 9-6.45 9-12V5l-9-4zm0 10.99h7c-.53 4.12-3.28 7.79-7 8.94V12H5V6.3l7-3.11v8.8z"/>
          </svg>
          <span>Secure Connection</span>
        </div>

        <div class="form-group">
          <label for="password">Authentication Code</label>
          <input
            type="password"
            id="password"
            bind:value={password}
            placeholder="Enter your access code"
            disabled={isLoading}
          />
        </div>

        {#if loginError}
          <div class="error-message">
            <svg viewBox="0 0 24 24" width="16" height="16" fill="#d93025">
              <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm1 15h-2v-2h2v2zm0-4h-2V7h2v6z"/>
            </svg>
            {loginError}
          </div>
        {/if}

        <button
          class="login-btn"
          on:click={handleLogin}
          disabled={isLoading || !password}
        >
          {#if isLoading}
            <span class="spinner"></span>
            Verifying...
          {:else}
            Authenticate
          {/if}
        </button>

        <p class="login-help">
          Forgot your access code? Contact your system administrator.
        </p>
      </div>

      <div class="login-footer">
        <p>© 2024 Corporate Systems Inc. All rights reserved.</p>
        <p class="footer-links">
          <span>Privacy Policy</span> · <span>Terms of Service</span> · <span>Help</span>
        </p>
      </div>
    </div>

  {:else if state === 'loading'}
    <!-- FAKE LOADING SCREEN - The troll moment -->
    <div class="fake-loading-screen">
      <img src={LOADING_IMAGE} alt="Loading..." class="loading-troll-image" />
      <div class="fake-progress-container">
        <p class="fake-loading-text">Initializing system modules...</p>
        <div class="fake-progress-bar">
          <div class="fake-progress-fill"></div>
        </div>
        <p class="fake-loading-subtext">Please wait while we verify your credentials</p>
      </div>
    </div>

  {:else}
    <!-- Floating background hearts (only shown after login) -->
    {#each hearts as heart}
      <div
        class="floating-heart"
        style="
          left: {heart.x}%;
          animation-delay: {heart.delay}s;
          animation-duration: {heart.duration}s;
          font-size: {heart.size}px;
        "
      >
        ❤️
      </div>
    {/each}
  {/if}

  {#if state === 'asking'}
    <div class="card">
      <h1 class="title">Lilian, will you be my valentine?</h1>

      <!-- GIF Slideshow -->
      <div class="slideshow-container">
        <button class="slide-btn prev" on:click={prevSlide}>❮</button>

        {#key currentSlide}
          <div class="slide">
            <img src={SLIDESHOW_IMAGES[currentSlide].url} alt="Slide {currentSlide + 1}" />
            {#if SLIDESHOW_IMAGES[currentSlide].text}
              <div class="slide-text">{SLIDESHOW_IMAGES[currentSlide].text}</div>
            {/if}
          </div>
        {/key}

        <button class="slide-btn next" on:click={nextSlide}>❯</button>

        <div class="slide-dots">
          {#each SLIDESHOW_IMAGES as _, i}
            <span
              class="dot"
              class:active={i === currentSlide}
              on:click={() => goToSlide(i)}
              on:keydown={(e) => e.key === 'Enter' && goToSlide(i)}
              role="button"
              tabindex="0"
            ></span>
          {/each}
        </div>
      </div>

      <div class="buttons">
        <button
          class="yes-btn"
          on:click={handleYesClick}
        >
          YES! 💕
        </button>

        <button
          class="no-btn"
          class:moved={noButtonMoved}
          class:tired={tirednessLevel >= 1}
          class:very-tired={tirednessLevel >= 2}
          class:exhausted={tirednessLevel >= 4}
          class:given-up={hasGivenUp}
          style={noButtonMoved ? `
            position: fixed;
            left: ${noButtonPosition.x}px;
            top: ${noButtonPosition.y}px;
            transition: left ${transitionSpeed}s, top ${transitionSpeed}s;
          ` : ''}
          on:mouseenter={moveNoButton}
          on:click={handleNoClick}
        >
          {noButtonText}
        </button>
      </div>

      {#if tirednessLevel >= 1 && tirednessLevel < 5}
        <p class="tired-hint">The No button is getting tired... maybe just say yes? 😏</p>
      {:else if hasGivenUp}
        <p class="tired-hint">Fine, click it if you must... but you won't! 💕</p>
      {/if}
    </div>

  {:else if state === 'wrong-answer'}
    <div class="fullscreen-meme">
      <!-- Stitch in the rain - 5 seconds -->
      <img src={WRONG_ANSWER_GIF} alt="Wrong Answer" class="big-meme-image sixty-percent" />
    </div>

  {:else if state === 'second-meme'}
    <div class="fullscreen-meme">
      <!-- Fumble GIF - 5 seconds -->
      <img src={SECOND_MEME_GIF} alt="Fumble" class="big-meme-image" />
    </div>

  {:else if state === 'try-again'}
    <!-- Try again overlay on top of the main page -->
    <div class="card">
      <h1 class="title">Lilian, will you be my valentine?</h1>
      <div class="try-again-overlay">
        <img src={TRY_AGAIN_GIF} alt="Try Again" class="try-again-gif" />
      </div>
    </div>

  {:else if state === 'success-gif1'}
    <!-- First success GIF - Dog meme with fade in -->
    <div class="success-gif-container fade-in">
      <img src={SUCCESS_GIF_1} alt="Yes!" class="success-gif" />
    </div>

  {:else if state === 'success-gif2'}
    <!-- Second success GIF - Dwight win -->
    <div class="success-gif-container fade-in">
      <img src={SUCCESS_GIF_2} alt="Winner!" class="success-gif" />
    </div>

  {:else if state === 'success'}
    <div class="success-container">
      <!-- Celebration hearts -->
      {#each celebrationHearts as heart}
        <div
          class="celebration-heart"
          style="
            left: {heart.x}%;
            top: {heart.y}%;
            transform: rotate({heart.rotation}deg);
            animation-delay: {Math.random() * 0.5}s;
          "
        >
          💕
        </div>
      {/each}

      <div class="success-content fade-in">
        <h1 class="success-title">YAAAY! 🎉💕</h1>
        <p class="success-message">I knew you'd say yes!</p>

        <!-- Your special photo -->
        <img src={FINAL_PHOTO} alt="Us!" class="final-photo" />

        <div class="personal-message">
          <p>join me for a movie and Omakase? 🍣❤️</p>
        </div>

        <div class="hearts-row">
          ❤️ 🧡 💛 💚 💙 💜 🖤 🤍 🤎 💕
        </div>
      </div>
    </div>
  {/if}
</main>

<style>
  /* LOGIN PAGE STYLES - Corporate/Boring look */
  .login-page,
  .loading-page {
    background: #f8f9fa !important;
  }

  .login-container {
    width: 100%;
    max-width: 420px;
    padding: 2rem;
    font-family: 'Segoe UI', -apple-system, BlinkMacSystemFont, sans-serif;
  }

  .login-header {
    text-align: center;
    margin-bottom: 2rem;
  }

  .company-logo {
    margin-bottom: 1rem;
  }

  .login-title {
    font-size: 1.5rem;
    font-weight: 600;
    color: #202124;
    margin: 0 0 0.5rem 0;
    font-family: 'Segoe UI', -apple-system, BlinkMacSystemFont, sans-serif;
  }

  .login-subtitle {
    font-size: 0.875rem;
    color: #5f6368;
    margin: 0;
  }

  .login-card {
    background: white;
    border-radius: 8px;
    padding: 2rem;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.12), 0 1px 2px rgba(0, 0, 0, 0.24);
  }

  .security-badge {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    font-size: 0.75rem;
    color: #5f6368;
    margin-bottom: 1.5rem;
    padding: 0.5rem;
    background: #f8f9fa;
    border-radius: 4px;
  }

  .form-group {
    margin-bottom: 1.5rem;
  }

  .form-group label {
    display: block;
    font-size: 0.875rem;
    font-weight: 500;
    color: #202124;
    margin-bottom: 0.5rem;
  }

  .form-group input {
    width: 100%;
    padding: 0.75rem 1rem;
    font-size: 1rem;
    border: 1px solid #dadce0;
    border-radius: 4px;
    outline: none;
    transition: border-color 0.2s;
    box-sizing: border-box;
  }

  .form-group input:focus {
    border-color: #1a73e8;
    box-shadow: 0 0 0 2px rgba(26, 115, 232, 0.2);
  }

  .form-group input:disabled {
    background: #f8f9fa;
    cursor: not-allowed;
  }

  .error-message {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    font-size: 0.875rem;
    color: #d93025;
    margin-bottom: 1rem;
    padding: 0.75rem;
    background: #fce8e6;
    border-radius: 4px;
  }

  .login-btn {
    width: 100%;
    padding: 0.875rem 1.5rem;
    font-size: 0.875rem;
    font-weight: 500;
    color: white;
    background: #1a73e8;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    transition: background 0.2s;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 0.5rem;
    font-family: 'Segoe UI', -apple-system, BlinkMacSystemFont, sans-serif;
  }

  .login-btn:hover:not(:disabled) {
    background: #1557b0;
  }

  .login-btn:disabled {
    background: #dadce0;
    cursor: not-allowed;
  }

  .spinner {
    width: 16px;
    height: 16px;
    border: 2px solid rgba(255, 255, 255, 0.3);
    border-top-color: white;
    border-radius: 50%;
    animation: spin 0.8s linear infinite;
  }

  @keyframes spin {
    to { transform: rotate(360deg); }
  }

  .login-help {
    font-size: 0.75rem;
    color: #5f6368;
    text-align: center;
    margin: 1rem 0 0 0;
  }

  .login-footer {
    text-align: center;
    margin-top: 2rem;
    font-size: 0.75rem;
    color: #5f6368;
  }

  .login-footer p {
    margin: 0.25rem 0;
  }

  .footer-links span {
    cursor: pointer;
  }

  .footer-links span:hover {
    text-decoration: underline;
  }

  /* FAKE LOADING SCREEN */
  .fake-loading-screen {
    width: 100%;
    height: 100vh;
    background: #f8f9fa;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    gap: 2rem;
    font-family: 'Segoe UI', -apple-system, BlinkMacSystemFont, sans-serif;
  }

  .loading-troll-image {
    width: 200px;
    height: auto;
    border-radius: 8px;
  }

  .fake-progress-container {
    text-align: center;
    width: 300px;
  }

  .fake-loading-text {
    font-size: 0.875rem;
    color: #202124;
    margin: 0 0 1rem 0;
  }

  .fake-progress-bar {
    width: 100%;
    height: 4px;
    background: #dadce0;
    border-radius: 2px;
    overflow: hidden;
  }

  .fake-progress-fill {
    height: 100%;
    background: #1a73e8;
    animation: progress-animation 3s ease-in-out forwards;
  }

  @keyframes progress-animation {
    0% { width: 0%; }
    20% { width: 25%; }
    50% { width: 60%; }
    80% { width: 85%; }
    100% { width: 100%; }
  }

  .fake-loading-subtext {
    font-size: 0.75rem;
    color: #5f6368;
    margin: 1rem 0 0 0;
  }

  /* VALENTINE STYLES */
  main {
    background: linear-gradient(135deg, #ff6b9d 0%, #ff8a80 50%, #ffab91 100%);
    min-height: 100vh;
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    position: relative;
    overflow: hidden;
  }

  /* Floating hearts background */
  .floating-heart {
    position: fixed;
    bottom: -50px;
    animation: float-up linear infinite;
    opacity: 0.6;
    pointer-events: none;
    z-index: 0;
  }

  @keyframes float-up {
    0% {
      transform: translateY(0) rotate(0deg);
      opacity: 0.6;
    }
    100% {
      transform: translateY(-110vh) rotate(360deg);
      opacity: 0;
    }
  }

  /* Main card */
  .card {
    background: rgba(255, 255, 255, 0.95);
    border-radius: 30px;
    padding: 3rem 4rem;
    box-shadow: 0 20px 60px rgba(0, 0, 0, 0.15);
    text-align: center;
    z-index: 10;
    max-width: 500px;
    margin: 1rem;
  }

  .title {
    font-family: 'Dancing Script', cursive;
    font-size: 3rem;
    color: #e91e63;
    margin-bottom: 1.5rem;
    text-shadow: 2px 2px 4px rgba(233, 30, 99, 0.2);
  }

  /* Slideshow */
  .slideshow-container {
    position: relative;
    width: 280px;
    margin: 1.5rem auto;
    border-radius: 20px;
    overflow: hidden;
    background: #000;
    box-shadow: 0 8px 25px rgba(0, 0, 0, 0.2);
  }

  .slide {
    position: relative;
    width: 100%;
    aspect-ratio: 1;
    display: flex;
    justify-content: center;
    align-items: center;
    overflow: hidden;
  }

  .slide img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  .slide-text {
    position: absolute;
    bottom: 15px;
    left: 50%;
    transform: translateX(-50%);
    background: rgba(0, 0, 0, 0.7);
    color: white;
    padding: 8px 16px;
    border-radius: 20px;
    font-size: 1rem;
    font-weight: 600;
    text-align: center;
    white-space: nowrap;
    text-shadow: 0 2px 4px rgba(0, 0, 0, 0.5);
  }

  .slide-btn {
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    background: rgba(255, 255, 255, 0.8);
    border: none;
    width: 32px;
    height: 32px;
    border-radius: 50%;
    cursor: pointer;
    font-size: 1rem;
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 10;
    transition: all 0.2s;
  }

  .slide-btn:hover {
    background: white;
    transform: translateY(-50%) scale(1.1);
  }

  .slide-btn.prev {
    left: 8px;
  }

  .slide-btn.next {
    right: 8px;
  }

  .slide-dots {
    position: absolute;
    bottom: 8px;
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    gap: 6px;
    z-index: 10;
  }

  .dot {
    width: 8px;
    height: 8px;
    border-radius: 50%;
    background: rgba(255, 255, 255, 0.5);
    cursor: pointer;
    transition: all 0.2s;
  }

  .dot:hover {
    background: rgba(255, 255, 255, 0.8);
  }

  .dot.active {
    background: white;
    transform: scale(1.2);
  }

  /* Buttons container */
  .buttons {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 2rem;
    margin-top: 2rem;
    flex-wrap: wrap;
  }

  /* YES button - THE BIG ONE */
  .yes-btn {
    background: linear-gradient(135deg, #ff6b9d 0%, #e91e63 100%);
    color: white;
    border: none;
    padding: 1.5rem 3rem;
    font-size: 1.8rem;
    font-weight: 700;
    border-radius: 50px;
    cursor: pointer;
    box-shadow: 0 10px 30px rgba(233, 30, 99, 0.4);
    transition: all 0.3s ease;
    font-family: 'Poppins', sans-serif;
  }

  .yes-btn:hover {
    transform: scale(1.1) !important;
    box-shadow: 0 15px 40px rgba(233, 30, 99, 0.5);
  }

  /* NO button */
  .no-btn {
    background: #f5f5f5;
    color: #999;
    border: 2px solid #ddd;
    padding: 0.5rem 1.5rem;
    font-size: 0.9rem;
    border-radius: 25px;
    cursor: pointer;
    transition: all 0.3s ease;
    font-family: 'Poppins', sans-serif;
  }

  .no-btn:hover {
    background: #eee;
  }

  .no-btn.moved {
    z-index: 1000;
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  }

  .no-btn.tired {
    background: #fff0f0;
  }

  .no-btn.very-tired {
    animation: wobble 0.5s ease infinite;
    background: #ffe8e8;
  }

  .no-btn.exhausted {
    animation: shake-tired 0.3s ease infinite;
    background: #ffd0d0;
  }

  .no-btn.given-up {
    animation: none;
    background: #ffcccc;
    opacity: 0.8;
    cursor: pointer;
    /* Stays frozen in place - no position override */
  }

  @keyframes wobble {
    0%, 100% { transform: rotate(0deg); }
    25% { transform: rotate(-3deg); }
    75% { transform: rotate(3deg); }
  }

  @keyframes shake-tired {
    0%, 100% { transform: translateX(0) rotate(-3deg); }
    25% { transform: translateX(-2px) rotate(-5deg); }
    75% { transform: translateX(2px) rotate(-1deg); }
  }

  .tired-hint {
    margin-top: 1.5rem;
    color: #ff6b9d;
    font-size: 0.9rem;
    font-style: italic;
    animation: fadeIn 0.5s ease;
  }

  /* Fullscreen meme container - 80% of viewport */
  .fullscreen-meme {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    background: rgba(0, 0, 0, 0.9);
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    z-index: 100;
    animation: fadeIn 0.3s ease;
  }

  @keyframes fadeIn {
    from { opacity: 0; }
    to { opacity: 1; }
  }

  .wrong-title {
    font-size: 3rem;
    color: #ff1744;
    margin-bottom: 1rem;
    animation: pulse 0.5s infinite;
    text-shadow: 0 0 20px rgba(255, 23, 68, 0.5);
  }

  @keyframes pulse {
    0%, 100% { transform: scale(1); }
    50% { transform: scale(1.1); }
  }

  .big-meme-image {
    max-width: 80vw;
    max-height: 70vh;
    border-radius: 20px;
    box-shadow: 0 0 50px rgba(255, 107, 157, 0.5);
    object-fit: contain;
  }

  .big-meme-image.sixty-percent {
    max-width: 60vw;
    max-height: 60vh;
  }

  /* Try again overlay */
  .try-again-overlay {
    position: relative;
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 2rem auto;
  }

  .try-again-gif {
    width: 280px;
    max-width: 100%;
    border-radius: 20px;
    box-shadow: 0 8px 25px rgba(0, 0, 0, 0.2);
  }

  .meme-caption {
    color: white;
    font-size: 1.5rem;
    margin-top: 1.5rem;
    text-shadow: 0 2px 10px rgba(0, 0, 0, 0.5);
  }

  /* Success GIF sequence */
  .success-gif-container {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    background: linear-gradient(135deg, #ff6b9d 0%, #ff8a80 50%, #ffab91 100%);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 100;
  }

  .success-gif {
    max-width: 70vw;
    max-height: 70vh;
    border-radius: 20px;
    box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
    object-fit: contain;
  }

  .fade-in {
    animation: fadeInScale 0.5s ease forwards;
  }

  @keyframes fadeInScale {
    0% {
      opacity: 0;
      transform: scale(0.9);
    }
    100% {
      opacity: 1;
      transform: scale(1);
    }
  }

  .final-photo {
    max-width: 300px;
    max-height: 300px;
    border-radius: 20px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
    margin: 1rem 0;
    object-fit: cover;
  }

  /* Success screen */
  .success-container {
    position: relative;
    width: 100%;
    min-height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .celebration-heart {
    position: fixed;
    font-size: 2rem;
    animation: celebrate 2s ease-out forwards;
    pointer-events: none;
  }

  @keyframes celebrate {
    0% {
      opacity: 1;
      transform: scale(0);
    }
    50% {
      opacity: 1;
      transform: scale(1.5);
    }
    100% {
      opacity: 0;
      transform: scale(1) translateY(-100px);
    }
  }

  .success-content {
    background: rgba(255, 255, 255, 0.95);
    border-radius: 30px;
    padding: 3rem 4rem;
    box-shadow: 0 20px 60px rgba(0, 0, 0, 0.15);
    text-align: center;
    z-index: 10;
    animation: bounceIn 0.6s ease;
  }

  @keyframes bounceIn {
    0% { transform: scale(0.5); opacity: 0; }
    70% { transform: scale(1.1); }
    100% { transform: scale(1); opacity: 1; }
  }

  .success-title {
    font-family: 'Dancing Script', cursive;
    font-size: 4rem;
    color: #e91e63;
    margin-bottom: 1rem;
    animation: rainbow 2s linear infinite;
  }

  @keyframes rainbow {
    0% { color: #ff6b9d; }
    25% { color: #e91e63; }
    50% { color: #ff1744; }
    75% { color: #ff6b9d; }
    100% { color: #e91e63; }
  }

  .success-message {
    font-size: 1.5rem;
    color: #666;
    margin-bottom: 1.5rem;
  }

  .personal-message {
    background: linear-gradient(135deg, #fff5f7 0%, #ffe8ec 100%);
    border-radius: 15px;
    padding: 1.5rem;
    margin: 1.5rem 0;
    border: 2px dashed #ffc2d1;
  }

  .personal-message p {
    color: #e91e63;
    font-size: 1.1rem;
  }

  .hearts-row {
    font-size: 1.5rem;
    margin-top: 1rem;
    letter-spacing: 5px;
  }

  /* Responsive */
  @media (max-width: 600px) {
    .card, .success-content {
      padding: 2rem;
      margin: 1rem;
    }

    .title {
      font-size: 2rem;
    }

    .yes-btn {
      padding: 1rem 2rem;
      font-size: 1.4rem;
    }

    .success-title {
      font-size: 2.5rem;
    }
  }
</style>
