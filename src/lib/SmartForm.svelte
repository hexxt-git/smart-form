<script lang="ts">
	interface Input {
		name: string;
		description: string;
		condition: string;
		value: string;
		response: string;
		validate: (value: string) => boolean;
		required: boolean;
	}

	let inputs: Input[] = [
		{
			name: 'name',
			description: 'Enter your full name here',
			condition: 'user must enter a viable name',
			value: '',
			response: '',
			validate: (value: string) => {
				return value.length > 3;
			},
			required: true,
		},
		{
			name: 'email',
			description: 'Enter your email here',
			condition: 'user must enter a viable email address',
			value: '',
			response: '',
			validate: (value: string) => {
				return (
					value.match(
						/^[a-zA-Z0-9._-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,4}$/
					) !== null
				);
			},
			required: true,
		},
		{
			name: 'motivation',
			description: 'Enter your motivation for filling this form here',
			condition:
				'user must enter a motivation that is long enough that describes why they want to fill this form',
			value: '',
			response: '',
			validate: (value: string) => {
				return value.split(' ').length > 3;
			},
			required: true,
		},
		{
			name: 'password',
			description: 'Enter your password here',
			condition:
				'user must enter a password that is longer than 8 characters',
			value: '',
			response: '',
			validate: (value: string) => {
                // probably shouldnt let the ai see this
				return value.length > 8;
			},
			required: true,
		},
		{
			name: 'confirm password',
			description: 'Enter your password here',
			condition:
				'user must enter a password that is longer than 8 characters',
			value: '',
			response: '',
			validate: (value: string) => {
                // check if the password is the same as the previous password
				return value.length > 8;
			},
			required: true,
		},
	];

	import { GoogleGenerativeAI } from '@google/generative-ai';
	// move to the backend in the future to prevent API key exposure
	const genAI = new GoogleGenerativeAI(
		'AIzaSyBef_KLRpgxEjlYPCTGCED7z9KH-wE4yIc'
	); // this is a free key please dont steal it
	const model = genAI.getGenerativeModel({
		model: 'gemini-1.5-pro-latest', // the longer the system instruction the more in role the AI will be
        systemInstruction: `
            you are an assistant in charge of a form filling system. You are to assist users in filling out forms, deny and accept forms based on the information provided by the user. You are to provide feedback to the user based on the information they provide. You are to ensure that the user fills out the form correctly and provide feedback to the user based on the information they provide. your replys are very breif and always must stay friendly without going out of subject.
            you will be provided a json object with the user input and your response will depend on the is_valid key
            if no input is provided do not provide a response and never aknowledge that you need a JSON input.
        `,
	});

	const proccess = async (input: Input) => {
		if (input.value === '') return input;
		const prompt = `
        {
            "input name": "${input.name}",
            "description": "${input.description}",
            "condition": "${input.condition}",
            "user_input": "${input.value}",
            "is_valid": "${input.validate(input.value)}",
            "required": "${input.required}"
        }
        `;

		let success = false;
		let attempt = 0;
		while (!success && attempt < 3) {
			try {
				const response = await model.generateContent(prompt);
				input.response = response.response.text();
				success = true;
			} catch (e) {
				attempt++;
				await new Promise((resolve) => setTimeout(resolve, 5000));
			}
		}

		return input;
	};
	const proccessAll = () => {
		for (let index in inputs) {
			proccess(inputs[index]).then((res) => {
				inputs[index] = res;
			});
		}
	};
</script>

<form on:submit|preventDefault={proccessAll}>
	{#each inputs as input}
		<div class={input.validate(input.value) ? 'input' : 'input invalid'}>
			<input
				type="text"
				bind:value={input.value}
				placeholder={input.name}
                on:input={()=>input.response = ''}
			/>
			<!--on:input={()=>proccess(input)} /> disabled for performance -->
			<div class="response">
				{input.response}
			</div>
		</div>
	{/each}
	<button type="submit">Submit</button>
</form>

<style>
	form {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 10px;
        margin-top: 50px;
	}
	.input {
		background-color: #f0f0f0;
		color: black;
		min-width: 50%;
		width: 500px;
		max-width: 75%;
        border-radius: 5px;
	}
	.invalid .response{
        color: red;
	}
	input {
        display: block;
		border: none;
		background: transparent;
		border-bottom: dashed 1px #888;
		width: 100%;
        padding: 5px;
        font-size: inherit;
        box-sizing: border-box;
	}
	input:focus {
		outline: none;
	}
    .response{
        padding: 5px;
    }
    button {
        padding: 10px;
        border: none;
        background-color: #f0f0f0;
        border-radius: 5px;
        cursor: pointer;
        margin-top: 25px;
    }
</style>
