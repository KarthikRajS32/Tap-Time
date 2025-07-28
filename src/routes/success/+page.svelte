<script>
	import { onMount } from 'svelte';
	import { goto } from '$app/navigation';
	import { browser } from '$app/environment';
	import { v4 as uuidv4 } from 'uuid';

	let isLoading = true;
	let error = '';
    // Constants
    const isAlpha = /^[a-zA-Z\s]+$/;
    const apiUrlBase = 'https://1wwsjsc00f.execute-api.ap-south-1.amazonaws.com/test/customer';
    const firstSignupPageapiUrlBase = 'https://1wwsjsc00f.execute-api.ap-south-1.amazonaws.com/test/company';
    const cid = uuidv4();

    

	onMount(async () => {
		const initializeSignup = async () => {
			try {
				// Show loading overlay
				setTimeout(() => {
					isLoading = true;
				}, 300);

				

				// Create company and customer data
				await craeteFirstPageSignupAPiData();
				
				// Redirect to device access key page
				goto('/device');
				
			} catch (err) {
				console.error('Error during signup initialization:', err);
				
				// Redirect back to signup on error
				setTimeout(() => {
					goto('/register');
				}, 100);
			} finally {
				isLoading = false;
			}
		};

		initializeSignup();
	});

    async function craeteFirstPageSignupAPiData() {
        if (!browser) return;
        
        const firstSignupPageapiUrl = `${firstSignupPageapiUrlBase}/create`;
        const cname = localStorage.getItem('companyName');
        const clogo = localStorage.getItem('companyLogo');
        const companyStreet = localStorage.getItem('companyStreet');
        const companyCity = localStorage.getItem('companyCity');
        const companyState = localStorage.getItem('companyState');
        const companyZip = localStorage.getItem('companyZip');
        const NoOfDevices = localStorage.getItem('noOfDevices') || '';
        const NoOfEmployees = localStorage.getItem('noOfEmployees') || '';

        const userData = {
            CID: cid,
            CName: cname,
            CLogo: clogo,
            CAddress: `${companyStreet} -- ${companyCity} -- ${companyState} -- ${companyZip}`,
            NoOfDevices: NoOfDevices,
            NoOfEmployees: NoOfEmployees,
            ReportType: "Weekly",
            LastModifiedBy: 'Admin'
        };

        try {
            const response = await fetch(firstSignupPageapiUrl, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(userData)
            });

            if (!response.ok) {
                throw new Error(`Error: ${response.status}`);
            }

            const data = await response.json();
            if (!data.error) {
                createApiData();
            } else {
                setTimeout(() => {
                    goto('/register');
                }, 100);
            }
        } catch (err) {
            console.error('API error:', err);
            error = err instanceof Error ? err.message : 'An error occurred';
        }
    }

    function createApiData() {
        if (!browser) return;
        
        const customerId = uuidv4();
        const apiUrl = `${apiUrlBase}/create`;

        const firstName = localStorage.getItem('firstName');
        const lastName = localStorage.getItem('lastName');
        const address = localStorage.getItem('address');
        const phone = localStorage.getItem('phone');
        const email = localStorage.getItem('email');

        const userData = {
            CustomerID: customerId.toString(),
            CID: cid,
            FName: firstName,
            LName: lastName,
            Address: address,
            PhoneNumber: phone,
            Email: email,
            IsActive: true,
            LastModifiedBy: 'Admin'
        };

        fetch(apiUrl, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(userData)
        })
        .then(response => {
            if (!response.ok) {
                throw new Error(`Error: ${response.status}`);
            }
            return response.json();
        })
        .then(data => {
            goto('/login');
        })
        .catch(err => {
            console.error('Customer creation error:', err);
            error = err instanceof Error ? err.message : 'Customer creation failed';
        });
    }
</script>

<style>
	@keyframes spin {
		0% { transform: rotate(0deg); }
		100% { transform: rotate(360deg); }
	}

	.overlay {
		display: flex;
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		background-color: rgba(0, 0, 0, 0.5);
		z-index: 9999;
		justify-content: center;
		align-items: center;
	}

	.spinner {
		border: 4px solid #f3f3f3;
		border-top: 4px solid #02066F;
		border-radius: 50%;
		width: 40px;
		height: 40px;
		animation: spin 1s linear infinite;
	}

	.loading-container {
		display: flex;
		justify-content: center;
		align-items: center;
		height: 100vh;
	}

	.alert {
		padding: 0.75rem 1.25rem;
		margin-bottom: 1rem;
		border: 1px solid transparent;
		border-radius: 0.25rem;
	}

	.alert-danger {
		color: #721c24;
		background-color: #f8d7da;
		border-color: #f5c6cb;
	}
</style>

{#if isLoading}
	<div class="overlay">
		<div class="spinner"></div>
	</div>
{/if}

<div class="loading-container">
	<h2>Loading....</h2>
</div>

{#if error}
	<div class="alert alert-danger" role="alert">
		Error: {error}
	</div>
{/if}