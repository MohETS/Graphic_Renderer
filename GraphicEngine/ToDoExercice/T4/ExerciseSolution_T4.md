## 2. Change the color of the triangle to red.

#### File : modelclass.cpp

```c++
bool ModelClass::InitializeBuffers(ID3D11Device* device)
{
	...

	// Load the vertex array with data.
	vertices[0].position = XMFLOAT3(-1.0f, -1.0f, 0.0f);  // Bottom left.
	vertices[0].color = XMFLOAT4(1.0f, 0.0f, 0.0f, 1.0f); //Change the rgba value

	vertices[1].position = XMFLOAT3(0.0f, 1.0f, 0.0f);  // Top middle.
	vertices[1].color = XMFLOAT4(1.0f, 0.0f, 0.0f, 1.0f); //Change the rgba value

	vertices[2].position = XMFLOAT3(1.0f, -1.0f, 0.0f);  // Bottom right.
	vertices[2].color = XMFLOAT4(1.0f, 0.0f, 0.0f, 1.0f); //Change the rgba value

	...
}
```

## 3. Change the triangle to a square.

#### File : modelclass.cpp

```c++
bool ModelClass::InitializeBuffers(ID3D11Device* device)
{
	...

	// Set the number of vertices in the vertex array.
	m_vertexCount = 4; //Change the value to 4

	// Set the number of indices in the index array.
	m_indexCount = 6; //Change the value to 6

	...

	// Load the vertex array with data.
	vertices[0].position = XMFLOAT3(-1.0f, -1.0f, 0.0f);  // Bottom left.
	vertices[0].color = XMFLOAT4(0.0f, 1.0f, 0.0f, 1.0f);

	vertices[1].position = XMFLOAT3(-1.0f, 1.0f, 0.0f);  // Top left.
	vertices[1].color = XMFLOAT4(0.0f, 1.0f, 0.0f, 1.0f);

	vertices[2].position = XMFLOAT3(1.0f, -1.0f, 0.0f);  // Bottom right.
	vertices[2].color = XMFLOAT4(0.0f, 1.0f, 0.0f, 1.0f);

	//Add the new vertex which is the top right vertex
	vertices[3].position = XMFLOAT3(1.0f, 1.0f, 0.0f);  // Top right.
	vertices[3].color = XMFLOAT4(0.0f, 1.0f, 0.0f, 1.0f);

	// Load the index array with data.
	//Left triangle
	indices[0] = 0;  // Bottom left.
	indices[1] = 1;  // Top left.
	indices[2] = 2;  // Bottom right.

	//Add the new indices to form the new right triangle
	//Right triangle
	indices[3] = 1;  // Top left.
	indices[4] = 3;  // Top right.
	indices[5] = 2;  // Bottom right.

	...
}
```


## 4. Move the camera back 10 more units.

#### File : applicationclass.cpp

```c++
bool ApplicationClass::Initialize(int screenWidth, int screenHeight, HWND hwnd) {
	bool result;
	// Create and initialize the Direct3D object.
	m_Direct3D = new D3DClass;

	result = m_Direct3D->Initialize(screenWidth, screenHeight, VSYNC_ENABLED, hwnd, FULL_SCREEN, SCREEN_DEPTH, SCREEN_NEAR);
	if (!result)
	{
		MessageBox(hwnd, L"Could not initialize Direct3D", L"Error", MB_OK);
		return false;
	}

	// Create the camera object.
	m_Camera = new CameraClass;

	// Set the initial position of the camera.
	m_Camera->SetPosition(0.0f, 0.0f, -15.0f); //Change this value to add 10 more units in the back
	
	...
}
```

## 5. Change the pixel shader to output the color half as bright.

- ### Expected solution from the Tutorials

	#### File : color.ps

	```c++
	//////////////
	// TYPEDEFS //
	//////////////
	struct PixelInputType
	{
		float4 position : SV_POSITION;
		float4 color : COLOR;
	};


	////////////////////////////////////////////////////////////////////////////////
	// Pixel Shader
	////////////////////////////////////////////////////////////////////////////////
	float4 ColorPixelShader(PixelInputType input) : SV_TARGET
	{
		return input.color * 0.5f; // Multiply the input color by 0.5f (Half bright)
	}
	```

- ### Alternative solution
	This solution came about because I did not understand why I was unable to change the brightness of the rendered shape by setting the alpha value to 0.5f. Since the alpha value in an RGBA context handles *transparency*, the modification of the alpha value should have an effect on the brightness. After a lot of research, I found that the reason behind that "issue" was because Alpha Blending was not enabled in the graphics pipeline.

	Adding the following code to the Initialize method in the D3DClass will now enable you to modify the brightness of your shapes using the alpha value.

	```c++
	bool D3DClass::Initialize(int screenWidth, int screenHeight, bool vsync, HWND hwnd, bool fullscreen, float screenDepth, float screenNear){
		...

		D3D11_BLEND_DESC blendDesc = {};
		blendDesc.RenderTarget[0].BlendEnable = TRUE;
		blendDesc.RenderTarget[0].SrcBlend = D3D11_BLEND_SRC_ALPHA;
		blendDesc.RenderTarget[0].DestBlend = D3D11_BLEND_INV_SRC_ALPHA;
		blendDesc.RenderTarget[0].BlendOp = D3D11_BLEND_OP_ADD;
		blendDesc.RenderTarget[0].SrcBlendAlpha = D3D11_BLEND_ONE;
		blendDesc.RenderTarget[0].DestBlendAlpha = D3D11_BLEND_ZERO;
		blendDesc.RenderTarget[0].BlendOpAlpha = D3D11_BLEND_OP_ADD;
		blendDesc.RenderTarget[0].RenderTargetWriteMask = D3D11_COLOR_WRITE_ENABLE_ALL;

		ID3D11BlendState* blendState;
		m_device->CreateBlendState(&blendDesc, &blendState);

		float blendFactor[4] = { 0.0f, 0.0f, 0.0f, 0.0f };
		m_deviceContext->OMSetBlendState(blendState, blendFactor, 0xffffffff);

		return true;
	}
	```

	#### File : color.ps

	Then you can change the value of w (alpha in our RGBA context) to change the brightness

	```c++
	//////////////
	// TYPEDEFS //
	//////////////
	struct PixelInputType
	{
		float4 position : SV_POSITION;
		float4 color : COLOR;
	};


	////////////////////////////////////////////////////////////////////////////////
	// Pixel Shader
	////////////////////////////////////////////////////////////////////////////////
	float4 ColorPixelShader(PixelInputType input) : SV_TARGET
	{
		input.color.w = 0.5f; //Modify this value to change the brightness
		return input.color;
	}
	```

	However, when I was looking at the next tutorials to see if the blending concept would be brought up, I saw that tutorial 14: Font Engine would speak about the blending concept, so I would suggest just waiting to reach that tutorial, skip this method, and just go for the expected solution.





