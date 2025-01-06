## 2. Change the color of the light to green.

#### File : applicationclass.cpp

```c++
bool ApplicationClass::Initialize(int screenWidth, int screenHeight, HWND hwnd) {
	
	...

	// Create and initialize the light object.
	m_Light = new LightClass;

	m_Light->SetDiffuseColor(0.0f, 1.0f, 0.0f, 1.0f); //Change the RGBA value to get green
	m_Light->SetDirection(0.0f, 0.0f, 1.0f);

	return true;
}
```

## 3. Change the direction of the light to go down the positive and the negative X axis. You might want to change the speed of the rotation also.

#### File : applicationclass.cpp

```c++
bool ApplicationClass::Initialize(int screenWidth, int screenHeight, HWND hwnd) {
	
	...

	// Create and initialize the light object.
	m_Light = new LightClass;

	m_Light->SetDiffuseColor(1.0f, 1.0f, 1.0f, 1.0f); 
	m_Light->SetDirection(1.0f, 0.0f, 0.0f); //Change the x value to 1.0f for positive and -1.0f for negative

	return true;
}

...

bool ApplicationClass::Frame() {
	static float rotation = 0.0f;
	bool result;

	// Update the rotation variable each frame.
	rotation -= 0.0174532925f * 0.4f; //Change the value of the multiplier to speed up the rotation
	if (rotation < 0.0f)
	{
		rotation += 360.0f;
	}

	// Render the graphics scene.
	result = Render(rotation);
	if (!result)
	{
		return false;
	}
	return true;
}
```



