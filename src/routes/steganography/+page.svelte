<script>
	import { onMount } from 'svelte';
	import {fly, scale} from 'svelte/transition'

	let conceal = 0;
	let reveal = 0;
	let concealText = 0;
	let concealImage = 0;
	let revealText = 0;
	let revealImage = 0;
	let textConcealSubmitted = 0;
	let secretImageSubmitted = 0;
	let coverImageSubmitted = 0;
	let unreavealedImageSubmitted = 0;
	let coveredSecretImage;
	let buttonSecretImage;
	let buttonCoverImage;
	let buttonCoverText;
	let buttonRevealText;
	let buttonUnreavealedImage;
	let buttonFileSecretImage;
	let buttonFileCoverImage;
	let buttonFileCoverText;
	let buttonFileRevealText;
	let buttonFileUnreavealedImage;
	let buttonReavealedImage;
	let encryptionImageTextConcealSelected = 0;
	let decryptionImageTextConcealSelected = 0;
	let toBeConcealedText;
	let textCoverImage;
	let textRevealImage;
	let unreavealedImage;
	let secretImage;
	let coverImage;
	let concealedTextImage;
	let revealedTextImage;
	let textConcealProcessComplete = 0;
	let textRevealProcessComplete = 0;
	let imageConcealComplete = 0;
	let imageRevealComplete = 0;
	let exposedImage;
	let yesProcessAgain = 0;
	let noProcessAgain = 0;

	let readURLA = (input) =>  {
	    if (input.files && input.files[0]) {
	        var reader = new FileReader();
	        reader.onload = function (e) {
	        	document.querySelector("#encryption-image-conceal-text").src = e.target.result
	        	textCoverImage = e.target.result;
	        }
	        reader.readAsDataURL(input.files[0]);
	    }
	}

	let readURLB = (input) =>  {
	    if (input.files && input.files[0]) {
	        var reader = new FileReader();
	        reader.onload = function (e) {
	        	document.querySelector("#decryption-image-reveal-text").src = e.target.result
	        	textRevealImage = e.target.result;
	        }
	        reader.readAsDataURL(input.files[0]);
	    }
	}

	let readURLC = (input) =>  {
	    if (input.files && input.files[0]) {
	        var reader = new FileReader();
	        reader.onload = function (e) {
	        	document.querySelector("#secret-image").src = e.target.result
	        	secretImage = e.target.result;
	        }
	        reader.readAsDataURL(input.files[0]);
	    }
	}

	let readURLD = (input) =>  {
	    if (input.files && input.files[0]) {
	        var reader = new FileReader();
	        reader.onload = function (e) {
	        	document.querySelector("#cover-image").src = e.target.result
	        	coverImage = e.target.result;
	        }
	        reader.readAsDataURL(input.files[0]);
	    }
	}

	let readURLE = (input) =>  {
	    if (input.files && input.files[0]) {
	        var reader = new FileReader();
	        reader.onload = function (e) {
	        	document.querySelector("#unreavealed-image").src = e.target.result
	        	unreavealedImage = e.target.result;
	        }
	        reader.readAsDataURL(input.files[0]);
	    }
	}

	let hideText = () => {
		concealedTextImage = steg.encode(toBeConcealedText, textCoverImage);
	}

	let hideImage = () => {
		let tempSecretImage = new SimpleImage(secretImage);
		let tempCoverImage = new SimpleImage(coverImage);
		let combinedImage = new SimpleImage(300, 300);
		tempSecretImage.setSize(300,300)
		tempCoverImage.setSize(300,300)
		for (let i = 0; i < 300; i++) {
		    for (let j = 0; j < 300; j++) {
		      const secretImagePixel = tempSecretImage.getPixel(i, j);
		      const coverImagePixel = tempCoverImage.getPixel(i, j);
		      const combinedImagePixel = combinedImage.getPixel(i, j);
		      combinedImagePixel.setRed(
		        setCoverPixelValue(secretImagePixel.getRed()) / 16 + setCoverPixelValue(coverImagePixel.getRed())
		      );
		      combinedImagePixel.setGreen(
		        setCoverPixelValue(secretImagePixel.getGreen()) / 16 +
		          setCoverPixelValue(coverImagePixel.getGreen())
		      );
		      combinedImagePixel.setBlue(
		        setCoverPixelValue(secretImagePixel.getBlue()) / 16 +
		          setCoverPixelValue(coverImagePixel.getBlue())
		      );
		    }
		}
		// coveredSecretImage = combinedImage;
		let canvasCombinedImage = document.querySelector("#combined-image");
		combinedImage.drawTo(canvasCombinedImage);
		coveredSecretImage = canvasCombinedImage.toDataURL("image/png");
		console.log(coveredSecretImage);
	}

	let exposeImage = () => {
		let tempUnreavealedImage = new SimpleImage(unreavealedImage);
		let toBeReavealedImage = new SimpleImage(300, 300);
		tempUnreavealedImage.setSize(300,300)
		 for (let i = 0; i < 300; i++) {
		    for (let j = 0; j < 300; j++) {
		      const unreavealedImagePixel = tempUnreavealedImage.getPixel(i, j);
		      const toBeReavealedImagepixel = toBeReavealedImage.getPixel(i, j);
		      toBeReavealedImagepixel.setRed(
		        setExposePixelValue(unreavealedImagePixel.getRed()) 
		      );
		      toBeReavealedImagepixel.setGreen(
		        setExposePixelValue(unreavealedImagePixel.getGreen())
		      );
		      toBeReavealedImagepixel.setBlue(
		        setExposePixelValue(unreavealedImagePixel.getBlue()) 
		      );
		    }
		  }
		// coveredSecretImage = combinedImage;
		let canvasExposedImage = document.querySelector("#exposed-image");
		toBeReavealedImage.drawTo(canvasExposedImage);
		exposedImage = canvasExposedImage.toDataURL("image/png");
		console.log(exposedImage);
	}

	function setExposePixelValue(x){
		return ((x %16) * 16)+(x %16);
	}

	function setCoverPixelValue(x) {
		return x - (x % 16);
	}

	let exposeText = () => {
		revealedTextImage = steg.decode(textRevealImage);
	}

	let addImageCoverText = () => {
		buttonFileCoverText = document.querySelector("#file-cover-text");
    	buttonCoverText = document.querySelector("#btn-cover-text");
    	if (buttonCoverText) {
		     buttonCoverText.addEventListener("click", function(event) {
		     buttonFileCoverText.click();
		    })
	 	}
	}

	let addImageRevealText = () => {
		buttonFileRevealText = document.querySelector("#file-reveal-text");
    	buttonRevealText = document.querySelector("#btn-reveal-text");
    	if (buttonRevealText) {
		     buttonRevealText.addEventListener("click", function(event) {
		     buttonFileRevealText.click();
		   })
	 	}
	}

	let addSecretImage = () => {
		buttonFileSecretImage = document.querySelector("#file-secret-image");
    	buttonSecretImage = document.querySelector("#btn-secret-image");
    	if (buttonSecretImage) {
		     buttonSecretImage.addEventListener("click", function(event) {
		     buttonFileSecretImage.click();
		   })
	 	}
	}

	let addImageCover = () => {
		buttonFileCoverImage = document.querySelector("#file-cover-image");
    	buttonCoverImage = document.querySelector("#btn-cover-image");
    	if (buttonCoverImage) {
		     buttonCoverImage.addEventListener("click", function(event) {
		     buttonFileCoverImage.click();
		    })
	 	}
	}

	let addUnrevealedImage = () => {
		buttonFileUnreavealedImage = document.querySelector("#file-unreavealed-image");
    	buttonUnreavealedImage = document.querySelector("#btn-unreavealed-image");
    	if (buttonUnreavealedImage) {
		     buttonUnreavealedImage.addEventListener("click", function(event) {
		     buttonFileUnreavealedImage.click();
		    })
	 	}
	}
</script>

<style type="text/css">
	* {
		font-family: 'Anonymous Pro', monospace;
	}
</style>

<svelte:head>
	<title>Steganography</title>
	<meta name="description" content="Steganography Steganofy" />
</svelte:head>

<canvas class="d-none" id="combined-image"></canvas>
<canvas class="d-none" id="exposed-image"></canvas>

<div class="landing flex flex-direction-col flex-center-vertical flex-center-horizontal flex-gap-large">
	<div class="chat-container flex flex-direction-col flex-gap-regular">
		<div class="separator flex">
			<div class="w-50">
				<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600 }}>
					<div>What would you like to do?</div>
					<div class="flex flex-gap-regular">
						<button class="button-chat" on:click={()=>{conceal = 1}}>Conceal</button>
						<button class="button-chat" on:click={()=>{reveal = 1}}>Reveal</button>
					</div>
				</div>
			</div>
			<div class="w-50"></div>
		</div>
		{#if conceal == 1}
		<div class="separator flex">
			<div class="w-50"></div>
			<div class="w-50">
				<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600 }}>
					<div>I would like to conceal a message.</div>
				</div>
			</div>
		</div>
		<div class="separator flex">
			<div class="w-50">
				<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600, delay: 600 }}>
					<div>What message would you like to conceal?</div>
					<div class="flex flex-gap-regular">
						<button class="button-chat" on:click={()=>{concealText = 1}}>Text</button>
						<button class="button-chat" on:click={()=>{concealImage = 1}}>Image</button>
					</div>
				</div>
			</div>
			<div class="w-50"></div>
		</div>
		{/if}
		{#if reveal == 1}
		<div class="separator flex">
			<div class="w-50"></div>
			<div class="w-50">
				<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600 }}>
					<div>I would like to reveal a message.</div>
				</div>
			</div>
		</div>
		<div class="separator flex">
			<div class="w-50">
				<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600, delay: 600 }}>
					<div>What message would you like to reveal?</div>
					<div class="flex flex-gap-regular">
						<button class="button-chat" on:click={()=>{revealText = 1}}>Text</button>
						<button class="button-chat" on:click={()=>{revealImage = 1}}>Image</button>
					</div>
				</div>
			</div>
			<div class="w-50"></div>
		</div>
		{/if}
		{#if concealText == 1}
			<div class="separator flex">
				<div class="w-50"></div>
				<div class="w-50">
					<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600 }}>
						<div>A text message.</div>
					</div>
				</div>
			</div>
			<div class="separator flex">
				<div class="w-50">
					<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600, delay: 600 }}>
						<div>Kindly type the text on the given column below.</div>
						<textarea class="textarea" rows="8" placeholder="insert the text here" id="text"></textarea>
						<div class="flex">
							<button class="button-chat" on:click={() => {
								toBeConcealedText = document.querySelector("#text").value;
								textConcealSubmitted = 1;
							}}>Submit</button>
						</div>
					</div>
				</div>
				<div class="w-50"></div>
			</div>
		{/if}
		{#if concealImage == 1}
			<div class="separator flex">
				<div class="w-50"></div>
				<div class="w-50">
					<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600 }}>
						<div>An image.</div>
					</div>
				</div>
			</div>
			<div class="separator flex">
				<div class="w-50">
					<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600, delay: 600 }}>
						<div>Kindly upload the image that you want to hide. (upload an image with equal dimensions, e.g. 300x300)</div>
						<div class="flex flex-direction-col flex-gap-regular">
							<img src="" id="secret-image" class="image-show">
							<input type="file" hidden="hidden" id="file-secret-image" accept="image/*" on:change={() => {
			       				 readURLC(buttonFileSecretImage);
			       				 secretImageSubmitted = 1;
							}}>
							<div class="flex flex-gap-regular">
								<button class="button-chat" id="btn-secret-image" on:click={() => {addSecretImage()}}>Add Image</button>
							</div>
						</div>
					</div>
				</div>
				<div class="w-50"></div>
			</div>
		{/if}
		{#if revealText == 1}
			<div class="separator flex">
				<div class="w-50"></div>
				<div class="w-50">
					<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600 }}>
						<div>A text message.</div>
					</div>
				</div>
			</div>
			<div class="separator flex">
				<div class="w-50">
					<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600, delay: 600 }}>
						<div>Kindly upload the image embedded with the secret text.</div>
						<div class="flex flex-direction-col flex-gap-regular">
							<img src="" id="decryption-image-reveal-text" class="image-show">
							<input type="file" hidden="hidden" id="file-reveal-text" accept="image/*" on:change={() => {
			       				 readURLB(buttonFileRevealText);
			       				 decryptionImageTextConcealSelected = 1;
							}}>
							<div class="flex flex-gap-regular">
								<button class="button-chat" id="btn-reveal-text" on:click={() => {addImageRevealText()}}>Add Image</button>
							</div>
						</div>
					</div>
				</div>
				<div class="w-50"></div>
			</div>
		{/if}
		{#if revealImage == 1}
			<div class="separator flex">
				<div class="w-50"></div>
				<div class="w-50">
					<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600 }}>
						<div>An image.</div>
					</div>
				</div>
			</div>
			<div class="separator flex">
				<div class="w-50">
					<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600, delay: 600 }}>
						<div>Kindly upload the image embedded with the secret image.</div>
						<div class="flex flex-direction-col flex-gap-regular">
							<img src="" id="unreavealed-image" class="image-show">
							<input type="file" hidden="hidden" id="file-unreavealed-image" accept="image/*" on:change={() => {
			       				 readURLE(buttonFileUnreavealedImage);
			       				 unreavealedImageSubmitted = 1;
							}}>
							<div class="flex flex-gap-regular">
								<button class="button-chat" id="btn-unreavealed-image" on:click={() => {addUnrevealedImage()}}>Add Image</button>
							</div>
						</div>
					</div>
				</div>
				<div class="w-50"></div>
			</div>
		{/if}
		{#if textConcealSubmitted == 1}
			<div class="separator flex">
				<div class="w-50">
					<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600, delay: 600 }}>
						<div>Now, upload an image as the cover.</div>
						<div class="flex flex-direction-col flex-gap-regular">
							<img src="" id="encryption-image-conceal-text" class="image-show">
							<input type="file" hidden="hidden" id="file-cover-text" accept="image/*" on:change={() => {
			       				 readURLA(buttonFileCoverText);
			       				 encryptionImageTextConcealSelected = 1;
							}}>
							<div class="flex flex-gap-regular">
								<button class="button-chat" id="btn-cover-text" on:click={() => {addImageCoverText()}}>Add Image</button>
							</div>
						</div>
					</div>
				</div>
				<div class="w-50"></div>
			</div>
		{/if}
		{#if secretImageSubmitted == 1}
			<div class="separator flex">
				<div class="w-50">
					<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600, delay: 600 }}>
						<div>Now, upload an image as the cover. (upload an image with equal dimensions, e.g. 300x300)</div>
						<div class="flex flex-direction-col flex-gap-regular">
							<img src="" id="cover-image" class="image-show">
							<input type="file" hidden="hidden" id="file-cover-image" accept="image/*" on:change={() => {
			       				 readURLD(buttonFileCoverImage);
			       				 coverImageSubmitted = 1;
							}}>
							<div class="flex flex-gap-regular">
								<button class="button-chat" id="btn-cover-image" on:click={() => {addImageCover()}}>Add Image</button>
							</div>
						</div>
					</div>
				</div>
				<div class="w-50"></div>
			</div>
		{/if}
		{#if coverImageSubmitted == 1}
			<div class="separator flex">
				<div class="w-50">
					<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600, delay: 600 }}>
						<div>Excellent. Proceed with the images? (you can re-enter the images using the chat bubble above, click proceed when you are ready)</div>
						<div class="flex flex-direction-col flex-gap-regular">
							<div class="flex flex-gap-regular">
								<button class="button-chat" id="btn-cover-text" on:click={() => {hideImage();imageConcealComplete = 1; }}>Proceed</button>
							</div>
						</div>
					</div>
				</div>
				<div class="w-50"></div>
			</div>
		{/if}
		{#if unreavealedImageSubmitted == 1}
			<div class="separator flex">
				<div class="w-50">
					<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600, delay: 600 }}>
						<div>Excellent. Proceed with the image? (you can re-enter the images using the chat bubble above, click proceed when you are ready)</div>
						<div class="flex flex-direction-col flex-gap-regular">
							<div class="flex flex-gap-regular">
								<button class="button-chat" id="btn-reveal-image" on:click={() => {exposeImage(); imageRevealComplete = 1; }}>Proceed</button>
							</div>
						</div>
					</div>
				</div>
				<div class="w-50"></div>
			</div>
		{/if}
		{#if decryptionImageTextConcealSelected == 1}
			<div class="separator flex">
				<div class="w-50">
					<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600, delay: 600 }}>
						<div>Excellent. Proceed with the image? (you can re-enter the image using the chat bubble above, click proceed when you are ready)</div>
						<div class="flex flex-direction-col flex-gap-regular">
							<div class="flex flex-gap-regular">
								<button class="button-chat" id="btn-cover-text" on:click={() => {exposeText();textRevealProcessComplete = 1; }}>Proceed</button>
							</div>
						</div>
					</div>
				</div>
				<div class="w-50"></div>
			</div>
		{/if}
		{#if encryptionImageTextConcealSelected == 1}
			<div class="separator flex">
				<div class="w-50">
					<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600, delay: 600 }}>
						<div>Excellent. Proceed with the image and text? (you can re-enter the text and image using the chat bubble above, click proceed when you are ready)</div>
						<div class="flex flex-direction-col flex-gap-regular">
							<div class="flex flex-gap-regular">
								<button class="button-chat" id="btn-cover-text" on:click={() => {hideText();textConcealProcessComplete = 1; }}>Proceed</button>
							</div>
						</div>
					</div>
				</div>
				<div class="w-50"></div>
			</div>
		{/if}
		{#if textConcealProcessComplete == 1}
		<div class="separator flex">
				<div class="w-50"></div>
				<div class="w-50">
					<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600 }}>
						<div>Yes, please.</div>
					</div>
				</div>
			</div>
		<div class="separator flex">
			<div class="w-50">
				<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600, delay: 600 }}>
					<div>Here's the processed text and image. Your text has bee successfully concealed. You can download the image.</div>
					<div class="flex flex-direction-col flex-gap-regular">
						<img src="{concealedTextImage}" id="encrypted-text-image" class="image-show">
						<div class="flex flex-gap-regular">
							<a href="{concealedTextImage}" class="button-chat" download="">Download Image</a>
						</div>
					</div>
				</div>
			</div>
			<div class="w-50"></div>
		</div>
		<div class="separator flex">
			<div class="w-50">
				<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600, delay: 1200 }}>
					<div>Do you have anything else you want to process?</div>
					<div class="flex flex-gap-regular">
						<button class="button-chat" on:click={()=>{yesProcessAgain = 1}}>Yes</button>
						<button class="button-chat" on:click={()=>{noProcessAgain = 1}}>No</button>
					</div>
				</div>
			</div>
			<div class="w-50"></div>
		</div>
		{/if}
		{#if textRevealProcessComplete == 1}
			<div class="separator flex">
				<div class="w-50"></div>
				<div class="w-50">
					<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600 }}>
						<div>Yes, please.</div>
					</div>
				</div>
			</div>
			<div class="separator flex">
				<div class="w-50">
					<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600, delay: 600 }}>
						<div>Here's the text : </div>
						<div id="revealed-text">{revealedTextImage}</div>
						<div class="flex flex-direction-col flex-gap-regular">
							<img src="{concealedTextImage}" id="encrypted-text-image" class="image-show">
						</div>
					</div>
				</div>
				<div class="w-50"></div>
			</div>
			<div class="separator flex">
			<div class="w-50">
				<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600, delay: 1200 }}>
					<div>Do you have anything else you want to process?</div>
					<div class="flex flex-gap-regular">
						<button class="button-chat" on:click={()=>{yesProcessAgain = 1}}>Yes</button>
						<button class="button-chat" on:click={()=>{noProcessAgain = 1}}>No</button>
					</div>
				</div>
			</div>
			<div class="w-50"></div>
		</div>
		{/if}
		{#if imageConcealComplete == 1}
		<div class="separator flex">
			<div class="w-50"></div>
			<div class="w-50">
				<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600 }}>
					<div>Yes, please.</div>
				</div>
			</div>
		</div>
		<div class="separator flex">
			<div class="w-50">
				<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600, delay: 600 }}>
					<div>Here's the processed image. Your secret image has bee successfully concealed. You can download the image.</div>
					<div class="flex flex-direction-col flex-gap-regular">
						<img src="{coveredSecretImage}" class="image-show">
						<div class="flex flex-gap-regular">
							<a href="{coveredSecretImage}" class="button-chat" download="">Download Image</a>
						</div>
					</div>
				</div>
			</div>
			<div class="w-50"></div>
		</div>
		<div class="separator flex">
			<div class="w-50">
				<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600, delay: 600 }}>
					<div>Do you have anything else you want to process?</div>
					<div class="flex flex-gap-regular">
						<button class="button-chat" on:click={()=>{yesProcessAgain = 1}}>Yes</button>
						<button class="button-chat" on:click={()=>{noProcessAgain = 1}}>No</button>
					</div>
				</div>
			</div>
			<div class="w-50"></div>
		</div>
		{/if}
		{#if imageRevealComplete == 1}
		<div class="separator flex">
			<div class="w-50"></div>
			<div class="w-50">
				<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600 }}>
					<div>Yes, please.</div>
				</div>
			</div>
		</div>
		<div class="separator flex">
			<div class="w-50">
				<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600, delay: 600 }}>
					<div>Here's the processed image. Your secret image has bee successfully revealed. You can download the image.</div>
					<div class="flex flex-direction-col flex-gap-regular">
						<img src="{exposedImage}" class="image-show">
						<div class="flex flex-gap-regular">
							<a href="{exposedImage}" class="button-chat" download="">Download Image</a>
						</div>
					</div>
				</div>
			</div>
			<div class="w-50"></div>
		</div>
		<div class="separator flex">
			<div class="w-50">
				<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600, delay: 600 }}>
					<div>Do you have anything else you want to process?</div>
					<div class="flex flex-gap-regular">
						<button class="button-chat" on:click={()=>{yesProcessAgain = 1}}>Yes</button>
						<button class="button-chat" on:click={()=>{noProcessAgain = 1}}>No</button>
					</div>
				</div>
			</div>
			<div class="w-50"></div>
		</div>
		{/if}
		{#if yesProcessAgain == 1}
			<div class="separator flex">
				<div class="w-50"></div>
				<div class="w-50">
					<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600 }}>
						<div>Yes, I would like to do something else.</div>
					</div>
				</div>
			</div>
			<div class="separator flex">
				<div class="w-50">
					<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600 }}>
						<div>Okay, let me reboot the program for you. Please press this button to reboot.</div>
						<div class="flex flex-gap-regular">
							<button class="button-chat" on:click={()=>{
								conceal = 0;
								reveal = 0;
								concealText = 0;
								concealImage = 0;
								revealText = 0;
								revealImage = 0;
								textConcealSubmitted = 0;
								secretImageSubmitted = 0;
								coverImageSubmitted = 0;
								unreavealedImageSubmitted = 0;
								decryptionImageTextConcealSelected = 0;
								encryptionImageTextConcealSelected = 0;
								textConcealProcessComplete = 0;
								textRevealProcessComplete = 0;
								imageConcealComplete = 0;
								imageRevealComplete = 0;
								yesProcessAgain = 0;
							}}>Reboot</button>
						</div>
					</div>
				</div>
				<div class="w-50"></div>
			</div>
		{/if}
		{#if noProcessAgain == 1}
			<div class="separator flex">
				<div class="w-50"></div>
				<div class="w-50">
					<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600 }}>
						<div>No, I would like to quit the program.</div>
					</div>
				</div>
			</div>
			<div class="separator flex">
				<div class="w-50">
					<div class="chat-bubble flex flex-direction-col flex-gap-semi-large" in:fly={{ y: -20, duration: 600 }}>
						<div>Okay. Thank you, and stay private.</div>
					</div>
				</div>
				<div class="w-50"></div>
			</div>
		{/if}
	</div>
</div>
<div class="footer flex flex-center-horizontal flex-center-vertical">Made by Farhan & Riandy. Starts With Bismillah & Ends In Alhamdulillah.</div>
