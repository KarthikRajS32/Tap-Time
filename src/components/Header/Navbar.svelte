<script lang="ts">

import { page } from '$app/stores';
import { onMount } from 'svelte';
import { get } from 'svelte/store';
import { createEventDispatcher } from 'svelte';

let sidebarOpen = false;
let dropdownOpen = false;
let currentPath = '';
let showModal = false;
let showHomeModal = false; // State for home modal
let userType = "";

onMount(() => {
	currentPath = get(page).url.pathname;
	userType = localStorage.getItem('adminType') || '';
});

$: currentPath = $page.url.pathname; // reactive update


function isActive(path: string) {
	return currentPath === path;
}

	function toggleSidebar() {
		sidebarOpen = !sidebarOpen;
		
	}
	

	function toggleDropdown() {
		dropdownOpen = !dropdownOpen;
	}

	

const dispatch = createEventDispatcher();


function logOutAction() {
	if (showModal=true) {
		sidebarOpen=false
	}
//   showModal = true;
}

function close() {
  showModal = false;
}

function HomePage() {
  showHomeModal = true; 
}

function handleLogout() {
  localStorage.removeItem("username");
  localStorage.removeItem("companyID");
  localStorage.removeItem("customId");
  localStorage.removeItem("password");
  localStorage.removeItem("adminType");

  setTimeout(() => {
	window.location.href = "/login"; // or use routing if needed
  }, 10);
}

</script>

<!-- Header -->
<header class="fixed top-0 left-0 right-0 z-50 bg-white border-b border-[#02066F]">
	<div class="flex justify-between items-center h-[70px] px-4">
		<!-- Logo -->
		<div class="flex items-center gap-2">
			<img src="/icode-logo.png" alt="icode-logo" class="w-20 cursor-pointer" on:click={HomePage}/>
		</div>

		<!-- Desktop Nav -->
		<nav class="hidden lg:flex items-center gap-10 text-gray-500 text-lg">
			{#if userType !== 'Admin'}
			<a href="/device" 
      class={isActive('/device')
		? 'text-[#02066F] font-semibold underline decoration-2 underline-offset-12 underline-offset-4'
		: 'text-gray-500 underline-offset-4 hover:underline hover:decoration-2 hover:underline-offset-12 focus:text-[#02066F] focus:underline focus:decoration-2 focus:underline-offset-12'}
    >Device</a>
	{/if}
			<a href="/employeelist" 
      class={isActive('/employeelist')
		? 'text-[#02066F] font-semibold underline decoration-2 underline-offset-12 underline-offset-4'
		: 'text-gray-500 underline-offset-4 hover:underline hover:decoration-2 hover:underline-offset-12 focus:text-[#02066F] focus:underline focus:decoration-2 focus:underline-offset-12'}
    >Employee Management</a>

			<!-- Dropdown -->
			<div class="relative">
				<button
        class={isActive('/reportsummary') || isActive('/daywisereport') || isActive('/salariedreport') || isActive('/reportsetting')
        ? 'text-[#02066F] font-semibold underline decoration-2 underline-offset-12 underline-offset-4 cursor-pointer'
        : 'text-gray-500 underline-offset-4 hover:underline hover:decoration-2 hover:underline-offset-12 cursor-pointer focus:text-[#02066F] focus:underline focus:decoration-2 focus:underline-offset-12'}
        
					on:click={toggleDropdown}
				>
					Report
				</button>
				{#if dropdownOpen}
					<ul class="absolute mt-2 w-48 bg-white shadow-md border rounded z-10 text-[#02066F]">
						<li><a href="/reportsummary" class="block px-4 py-2 hover:bg-gray-100">Report Summary</a></li>
						<li><a href="/reportsetting" class="block px-4 py-2 hover:bg-gray-100">Report Settings</a></li>
					</ul>
				{/if}
			</div>

			<a href="/profile" 
      class={isActive('/profile')
		? 'text-[#02066F] font-semibold underline decoration-2 underline-offset-12 underline-offset-4'
		: 'text-gray-500 underline-offset-4 hover:underline hover:decoration-2 hover:underline-offset-12 focus:text-[#02066F] focus:underline focus:decoration-2 focus:underline-offset-12'}
    >Profile</a>
			<a href="/contact" 
      class={isActive('/contact')
		? 'text-[#02066F] font-semibold underline decoration-2 underline-offset-12 underline-offset-4'
		: 'text-gray-500 underline-offset-4 hover:underline hover:decoration-2 hover:underline-offset-12 focus:text-[#02066F] focus:underline focus:decoration-2 focus:underline-offset-12'}
    >Contact Us</a>
			
	
	<!-- Logout Button -->
<button on:click={logOutAction} class="px-4 py-2 font-semibold bg-[#02066F] text-white rounded-xl hover:opacity-85 cursor-pointer transition-colors">
	Logout
  </button>
  
  <!-- Logout Modal -->
  {#if showModal}
	<div class="fixed inset-0 z-50 flex items-center justify-center px-4"
	style="background: rgba(0, 0, 0, 0.5)">
	  <div class="bg-white rounded-lg w-auto max-w-md shadow-lg overflow-hidden">
		<!-- Header -->
		<div class="bg-[#02066F] text-white p-1 flex justify-between text-center items-center">
		  <h2 class="text-xl font-semibold w-full text-center">Logout</h2>
		  <button class="text-gray-400 hover:text-white text-4xl cursor-pointer p-2" on:click={close}>&times;</button>
		</div>
  
		<!-- Body -->
		<div class="p-6 text-center">
		  <h5 class="text-xl font-bold mb-4 text-gray-800">Are you sure you want to logout?</h5>
		  <div class="flex justify-center space-x-2">
			<button class="bg-[#02066F] opacity-80 hover:opacity-70 text-white px-6 py-2 rounded-md cursor-pointer" on:click={handleLogout}>Yes</button>
			<button class="bg-white text-black px-6 py-2 rounded-md cursor-pointer border-1 border-[#02066F]" on:click={close}>No</button>
		  </div>
		</div>
	  </div>
	</div>
  {/if}

		</nav>

		<!-- Mobile Menu Button -->
		<button class="lg:hidden text-[#02066F]" on:click={toggleSidebar} aria-label="Toggle sidebar">
			<svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
				<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16" />
			</svg>
			  <!-- Logout Modal -->
			  {#if showModal}
				<div class="fixed inset-0 z-50 flex items-center justify-center px-4"
				style="background: rgba(0, 0, 0, 0.5)">
				  <div class="bg-white rounded-lg shadow-xl w-full max-w-sm overflow-hidden">
							  
					<!-- Header -->
					<div class="bg-[#02066F] text-white p-4 flex justify-between text-center items-center">
					  <h2 class="text-xl font-semibold w-full text-center ">Logout</h2>
					  <button class="text-gray-400 hover:text-white text-4xl cursor-pointer p-2" on:click={close}>&times;</button>
					</div>
			  
					<!-- Body -->
					<div class="p-6 text-center">
					  <h5 class="text-center text-sm md:text-xl font-bold mb-6 text-gray-800">Are you sure, you want to logout?</h5>
					  <div class="flex justify-center space-x-2">
						<button class="bg-[#02066F] opacity-80 hover:opacity-60 text-white px-6 py-2 rounded font-semibold cursor-pointer transition" on:click={handleLogout}>Yes</button>
						<button class="bg-white text-black px-5 py-2 rounded-md cursor-pointer border-1 border-[#02066F]" on:click={close}>No</button>
					  </div>
					</div>
				  </div>
				</div>
			  {/if}
		</button>
		
	</div>
</header>



<!-- Mobile Sidebar -->
{#if sidebarOpen}
	<div class="lg:hidden fixed inset-0 z-40" on:click={toggleSidebar}></div>

	<aside class="fixed top-0 left-0 h-full w-[250px] bg-[#02066F] z-50 shadow ">
		<div class="flex justify-between items-center p-2">
			<img class="w-16 pt-2" src="/logo.png" alt="logo" />
			<button on:click={toggleSidebar}>
				<svg class="w-6 h-6 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
					<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
				</svg>
			</button>
		</div>

		<nav class="space-y-3 text-[#02066F] font-medium p-10 pt-0">
			<a href="/device" 
				class={isActive('/device') 
					? 'block px-2 py-1 text-white rounded text-sm underline decoration-white decoration-2 underline-offset-6 underline-offset-4' 
					: 'block px-2 py-1 text-white rounded text-sm hover:underline hover:decoration-white hover:underline-offset-4'}
					
			>
				Device
			</a>
			
			<a href="/employeelist" 
				class={isActive('/employeelist') 
					? 'block px-2 py-1 text-white rounded text-sm underline decoration-white decoration-2 underline-offset-6 underline-offset-4' 
					: 'block px-2 py-1 text-white rounded text-sm hover:underline hover:decoration-white hover:underline-offset-4'}
			>
				Employee Management
			</a>
			
			<!-- Report Dropdown Section -->
			<div class="relative">
				<button
					class={isActive('/reportsummary') || isActive('/daywisereport') || isActive('/salariedreport') || isActive('/reportsetting')
						? 'w-full text-left px-2 py-1 text-white rounded font-medium text-sm underline decoration-white decoration-2 underline-offset-6 underline-offset-4'
						: 'w-full text-left px-2 py-1 text-white rounded font-medium text-sm hover:underline hover:decoration-white hover:underline-offset-4'}
					on:click={toggleDropdown}
				>
					Report
				</button>
				
				{#if dropdownOpen}
					<div class="pl-4 space-y-1 mt-1">
						<a href="/reportsummary" 
							class={isActive('/reportsummary') 
								? 'block px-2 py-1 text-white rounded text-sm underline decoration-white decoration-2 underline-offset-6 underline-offset-4' 
								: 'block px-2 py-1 text-white rounded text-sm hover:underline hover:decoration-white hover:underline-offset-4'}
						>
							Report Summary
						</a>
						<a href="/reportsetting" 
							class={isActive('/reportsetting') 
								? 'block px-2 py-1 text-white rounded text-sm underline decoration-white decoration-2 underline-offset-6 underline-offset-4' 
								: 'block px-2 py-1 text-white rounded text-sm hover:underline hover:decoration-white hover:underline-offset-4'}
						>
							Report Settings
						</a>
					</div>
				{/if}
			</div>
			
			<a href="/profile" 
				class={isActive('/profile') 
					? 'block px-2 py-1 text-white rounded text-sm underline decoration-white decoration-2 underline-offset-6 underline-offset-4' 
					: 'block px-2 py-1 text-white rounded text-sm hover:underline hover:decoration-white hover:underline-offset-4'}
			>
				Profile
			</a>
			
			<a href="/contact" 
				class={isActive('/contact') 
					? 'block px-2 py-1 text-white rounded text-sm underline decoration-white decoration-2 underline-offset-6 underline-offset-4' 
					: 'block px-2 py-1 text-white rounded text-sm hover:underline hover:decoration-white hover:underline-offset-4'}
			>
				Contact Us
			</a>
			
			<button class="w-25 mt-4 px-4 py-2 bg-white rounded text-center items-center justify-center" on:click={logOutAction}>
				Logout
			</button>
		</nav>
	</aside>
{/if}

 <!-- Home Modal  -->
{#if showHomeModal}
      <div class="fixed p-6 inset-0 flex items-center justify-center z-50"
	  style="background: rgba(0, 0, 0, 0.5)">
        <div class="bg-white rounded-lg shadow-xl w-full max-w-sm overflow-hidden">
          <div class="bg-[#02066F] text-white p-4 flex justify-between text-center items-center">
					  
            <h3 class="text-xl font-semibold w-full text-center">Home</h3>
			<button class="text-gray-400 hover:text-white text-4xl cursor-pointer p-2"  on:click={() => showHomeModal = false}>
				×</button>
          </div>
          <div class="p-4">
            <p class="text-center text-sm md:text-xl font-bold mb-6 text-gray-800">Are you sure, you want to go home?</p>
            <div class="flex justify-center space-x-4">
              <button 
                class="bg-[#02066F] opacity-80 hover:opacity-60 text-white px-6 py-2 rounded font-semibold cursor-pointer transition"
                on:click={() => window.location.href = '/'}
              >
                Yes
              </button>
              <button 
                class="border border-[#02066F] text-[#02066F] px-6 py-2 rounded font-semibold hover:bg-gray-100 cursor-pointer transition"
                on:click={() => showHomeModal = false}
              >
                No
              </button>
            </div>
          </div>
        </div>
      </div>
    {/if}