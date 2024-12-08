<script lang="ts">
	import { onMount } from "svelte"
    import axios from 'axios'   


    let clientId = '978fa928-7db7-4681-bf1c-c744f0512ab2' 
    let token: any
    let iss: string
    let launch: string
    let authorizationEndpoint: string
    let redirectUrl: string = 'http://localhost:5173/'
    let tokenEndpoint: string

    const constructAuthUrl = () => {
        const url = new URL(authorizationEndpoint)
        url.searchParams.set('client_id', '978fa928-7db7-4681-bf1c-c744f0512ab2')
        url.searchParams.set('redirect_uri', 'http://localhost:5173/')
        url.searchParams.set('scope', 'openid fhirUser launch user/Observation.read user/Observation.write user/Patient.read')
        url.searchParams.set('response_type', 'code')
        const iss = localStorage.getItem('issLocalStorage') ?? '';
        url.searchParams.set('aud', iss);
        const launch = localStorage.getItem('launchLocalStorage') ?? '';
        url.searchParams.set('launch', launch);

        // add code challenge param
        // add code cahllenge method param    
        console.log('Constructed URL:', url.href)
        
        // Redirect to the constructed URL
        window.location.href = url.href;
        
        return url.href  
    };

    const tokenRequest = async (code) => {
        const tokenEndpoint = localStorage.getItem('TokenEndStorage');
        const form = new URLSearchParams();
        form.set("grant_type", "authorization_code");
        form.set("code", code);
        form.set("redirect_uri", redirectUrl)
        form.set("client_id", clientId)
        const tokenResponse = await axios.post(tokenEndpoint, form, {            
        })




    }

	onMount(async () => {
        try {         
            const launchUrl = new URL(window.location.href);
            const issParam = launchUrl.searchParams.get('iss');
            if (issParam)
            localStorage.setItem('issLocalStorage', issParam ?? '');
            const launchParam = launchUrl.searchParams.get('launch');
            if (launchParam)
            localStorage.setItem('launchLocalStorage', launchParam?? '')
            const code = launchUrl.searchParams.get('code');
            console.log(code)          

        if (!issParam || !launchParam) {
            throw new Error('iss or launch parameters not found');
        }

        iss = issParam;
        launch = launchParam;

        const smartConfigurationResponse = await axios.get(`${iss}/.well-known/smart-configuration`);
        console.log('Smart Configuration Response:', smartConfigurationResponse.data); //log the response
        const smartConfiguration = smartConfigurationResponse.data;
        authorizationEndpoint = smartConfiguration.authorization_endpoint as string;
        tokenEndpoint = smartConfiguration.token_endpoint as string
        localStorage.setItem('TokenEndStorage', tokenEndpoint)
        localStorage.setItem('AuthEndStorage', authorizationEndpoint); 

        // redirectUrl = constructAuthUrl(launch)
        //console.log('Redirect URL:', redirectUrl); //log the redirect URL  

       // console.log('iss:', iss);
        //console.log('launch:', launch);
        //console.log('authorizationEndpoint:', authorizationEndpoint);
       // console.log('redirectUrl:', redirectUrl);
          
        } catch (error) {
            console.error('Error during redirect:', error);
        }
	});
	
</script>


<pre>
    {#if iss && launch && authorizationEndpoint}
        {JSON.stringify({iss, launch, authorizationEndpoint, redirectUrl}, null, 2)}
        <button on:click={() => constructAuthUrl(launch)}>Authenticate</button>
    {:else}
        Loading...
    {/if}
</pre>

