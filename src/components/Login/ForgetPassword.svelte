<script>
    import { onMount } from 'svelte';
    import emailjs from 'emailjs-com';
  
    let email = '';
    let otp = '';
    let generatedOtp = '';
    let showOtpSection = false;
    let errorMsg = '';
    let otpError = '';
  
    onMount(() => {
      emailjs.init('user_ID'); // Replace with your EmailJS user ID
    });
  
    function generateOTP() {
      let otp = '';
      const characters = '0123456789';
      for (let i = 0; i < 6; i++) {
        otp += characters.charAt(Math.floor(Math.random() * characters.length));
      }
      return otp;
    }
  
    function sendOtp() {
      const originalEmailValue = localStorage.getItem('email');
      if (email !== originalEmailValue) {
        errorMsg = 'This is the wrong email';
        return;
      }
      errorMsg = '';
      generatedOtp = generateOTP();
  
      const templateParams = {
        to_email: email,
        from_name: 'Tap-Time Support Team',
        subject: 'Password Reset OTP',
        message: `Your OTP for password reset is: ${generatedOtp}`
      };
  
      emailjs.send('service_nv0u86q', 'template_glkc6yl', templateParams)
        .then(() => {
          showOtpSection = true;
        })
        .catch(() => {
          errorMsg = 'Error sending email. Please try again later.';
        });
    }
  
    function verifyOtp() {
      if (otp === generatedOtp) {
        window.location.href = 'updatePassword.html';
      } else {
        otpError = 'Invalid OTP. Please try again.';
      }
    }
  </script>
  
 
  <div id="forgetPassword" class="flex flex-col md:flex-row min-h-screen ">
    <div class="md:w-1/2 bg-blue-100 flex flex-col justify-center items-center p-8">
      <img src="/loginImage.png" alt="Login" class="xl:w-113 md:w-42 w-60 md:pt-56 xl:mb-26 mb-24 xl:pt-42 pt-52" />
      <h3 class="text-center xl:text-3xl md:text-2xl text-[22px] md:pt-0 xl:pt-0 font-semibold text-gray-800 pb-2">Optimize Your Workforce Efficiency</h3>
      <p class="text-center text-lg md:text-base xl:text-base text-[15px] pb-12 text-gray-900">Streamlined time tracking and management for a productive workplace.</p>
    </div>
    
    <div class="md:w-1/2 flex justify-center items-center p-12 md:p-14 xl:pt-18 md:pt-30">
      <div class="w-full max-w-lg h-80 xl:h-80 md:h-90 bg-white p-12 xl:p-14 md:p-12 rounded-lg shadow-[0_0_10px_2px_rgba(0,0,0,0.1)]">
        <h2 class="text-center text-2xl xl:text-3xl md:text-3xl font-semibold mb-6 xl:mb-10 md:mb-8">Forget Password</h2>
        <div class="mb-4 xl:mb-4 md:mb-14 relative space-y-12 xl:space-y-0 md:space-y-0">
          <input id="email" type="email" bind:value={email} placeholder=" " required class="peer mt-1 block w-full px-3 py-3 md:px-4 border border-[#02066F] rounded-md shadow-sm focus:outline-none focus:ring-[#02066F] focus:border-[#02066F]" />
          <label for="email" class=" text-sm text-[#02066F] 
                  absolute left-4 top-2 bg-white px-1 transition-all
                  peer-placeholder-shown:top-2
                  peer-placeholder-shown:text-lg
                  peer-placeholder-shown:text-[#02066F]
                  peer-focus:top-[-10px]
                  peer-focus:text-sm
                  peer-focus:text-[#02066F]">Email</label>
          {#if errorMsg}
            <p class="text-red-500 text-base mt-1">{errorMsg}</p>
          {/if}
        </div>
        
        <button on:click={sendOtp} class="bg-[#02066F] cursor-pointer text-white w-full py-3 xl:py-3 md:py-3 text-xl rounded-md xl:mt-10">Submit</button>
  
        {#if showOtpSection}
          <div class="mt-6">
            <label for="otp" class="block text-sm font-medium text-gray-700">Enter OTP</label>
            <input id="otp" type="text" bind:value={otp} required class="mt-1 block w-full px-3 py-2 border border-blue-800 rounded-md shadow-sm focus:outline-none focus:ring-blue-800 focus:border-blue-800" />
            {#if otpError}
              <p class="text-red-500 text-sm mt-1">{otpError}</p>
            {/if}
            <button on:click={verifyOtp} class="mt-4 bg-[#02066F] text-white w-full py-2 rounded-md hover:bg-[#02066F]">Verify OTP</button>
          </div>
        {/if}
      </div>
    </div>
  </div>
  