## 3. Change the code to create two triangles that form a square. Map the entire texture to this square so that the entire texture shows up correctly on the screen.

#### File : modelclass.cpp

```c++
bool ModelClass::InitializeBuffers(ID3D11Device* device)
{
	...

	// Set the number of vertices in the vertex array.
	m_vertexCount = 4;

	// Set the number of indices in the index array.
	m_indexCount = 6;

    ...

	// Load the vertex array with data.
	vertices[0].position = XMFLOAT3(-1.0f, -1.0f, 0.0f);  // Bottom left.
	vertices[0].texture = XMFLOAT2(0.0f, 1.0f);

	vertices[1].position = XMFLOAT3(-1.0f, 1.0f, 0.0f);  // Top left.
	vertices[1].texture = XMFLOAT2(0.0f, 0.0f);

	vertices[2].position = XMFLOAT3(1.0f, -1.0f, 0.0f);  // Bottom right.
	vertices[2].texture = XMFLOAT2(1.0f, 1.0f);

	vertices[3].position = XMFLOAT3(1.0f, 1.0f, 0.0f);	// Top right.
	vertices[3].texture = XMFLOAT2(1.0f, 0.0f);

	// Load the index array with data.
	indices[0] = 0;  // Bottom left.
	indices[1] = 1;  // Top middle.
	indices[2] = 2;  // Bottom right.

	indices[3] = 1;  // Top left.
	indices[4] = 3;  // Top right.
	indices[5] = 2;  // Bottom right.

	...
}
```

## 6. Add a 24-bit Targa reader function so that your TextureClass automatically supports either 32 or 24-bit tga textures.

#### File : textureclass.h

```c++
class TextureClass
{

...

private:
	bool LoadTarga(char*); //Change  This line to have a general method
	bool LoadTarga32Bit(FILE*); //Change to have a method for 32bit
	bool LoadTarga24Bit(FILE*); //Change to have a method for 24bit

private:
	unsigned char* m_targaData;
	ID3D11Texture2D* m_texture;
	ID3D11ShaderResourceView* m_textureView;
	int m_width, m_height;
};
```


#### File : textureclass.cpp

```c++
bool TextureClass::LoadTarga(char* filename)
{
	int error, bpp;
	FILE* filePtr;
	unsigned int count;
	TargaHeader targaFileHeader;

	// Open the targa file for reading in binary.
	error = fopen_s(&filePtr, filename, "rb");
	if (error != 0)
	{
		return false;
	}

	// Read in the file header.
	count = (unsigned int)fread(&targaFileHeader, sizeof(TargaHeader), 1, filePtr);
	if (count != 1)
	{
		return false;
	}

	// Get the important information from the header.
	m_height = (int)targaFileHeader.height;
	m_width = (int)targaFileHeader.width;
	bpp = (int)targaFileHeader.bpp;

	// Checks if it is 32 bit or 24 bit. Calls the appropriate method
	if (bpp == 32) {
		return LoadTarga32Bit(filePtr);
	}

	if (bpp == 24) {
		return LoadTarga24Bit(filePtr);
	}

	return false;
}

/* This method handles the targa when its a 32bit .tga file */
bool TextureClass::LoadTarga32Bit(FILE* filePtr)
{
	int error, imageSize, index, i, j, k;
	unsigned int count;
	unsigned char* targaImage;

	// Calculate the size of the 32 bit image data.
	imageSize = m_width * m_height * 4;

	// Allocate memory for the targa image data.
	targaImage = new unsigned char[imageSize];

	// Read in the targa image data.
	count = (unsigned int)fread(targaImage, 1, imageSize, filePtr);
	if (count != imageSize)
	{
		return false;
	}

	// Close the file.
	error = fclose(filePtr);
	if (error != 0)
	{
		return false;
	}

	// Allocate memory for the targa destination data.
	m_targaData = new unsigned char[imageSize];

	// Initialize the index into the targa destination data array.
	index = 0;

	// Initialize the index into the targa image data.
	k = (m_width * m_height * 4) - (m_width * 4);

	// Now copy the targa image data into the targa destination array in the correct order since the targa format is stored upside down and also is not in RGBA order.
	for (j = 0; j < m_height; j++)
	{
		for (i = 0; i < m_width; i++)
		{
			m_targaData[index + 0] = targaImage[k + 2];  // Red.
			m_targaData[index + 1] = targaImage[k + 1];  // Green.
			m_targaData[index + 2] = targaImage[k + 0];  // Blue
			m_targaData[index + 3] = targaImage[k + 3];  // Alpha

			// Increment the indexes into the targa data.
			k += 4;
			index += 4;
		}

		// Set the targa image data index back to the preceding row at the beginning of the column since its reading it in upside down.
		k -= (m_width * 8);
	}

	// Release the targa image data now that it was copied into the destination array.
	delete[] targaImage;
	targaImage = 0;

	return true;
}

/* This method handles the targa when its a 24bit .tga file */
bool TextureClass::LoadTarga24Bit(FILE* filePtr)
{
	int error, imageSize, index, i, j, k;
	unsigned int count;
	unsigned char* targaImage;


	// Calculate the size of the 24 bit image data.
	imageSize = m_width * m_height * 3;

	// Allocate memory for the targa image data.
	targaImage = new unsigned char[imageSize];

	// Read in the targa image data.
	count = (unsigned int)fread(targaImage, 1, imageSize, filePtr);
	if (count != imageSize)
	{
		return false;
	}

	// Close the file.
	error = fclose(filePtr);
	if (error != 0)
	{
		return false;
	}

	// Allocate memory for the targa destination data.
	m_targaData = new unsigned char[m_width * m_height * 4];

	// Initialize the index into the targa destination data array.
	index = 0;

	// Initialize the index into the targa image data.
	k = (m_width * m_height * 3) - (m_width * 3);

	// Now copy the targa image data into the targa destination array in the correct order since the targa format is stored upside down and also is not in RGBA order.
	for (j = 0; j < m_height; j++)
	{
		for (i = 0; i < m_width; i++)
		{
			m_targaData[index + 0] = targaImage[k + 2];  // Red.
			m_targaData[index + 1] = targaImage[k + 1];  // Green.
			m_targaData[index + 2] = targaImage[k + 0];  // Blue
			m_targaData[index + 3] = 255;  // Alpha

			// Increment the indexes into the targa data.
			k += 3;
			index += 4;
		}

		// Set the targa image data index back to the preceding row at the beginning of the column since its reading it in upside down.
		k -= (m_width * 6);
	}

	// Release the targa image data now that it was copied into the destination array.
	delete[] targaImage;
	targaImage = 0;

	return true;
}
```



