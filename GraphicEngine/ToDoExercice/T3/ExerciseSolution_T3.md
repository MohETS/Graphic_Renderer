## 3. Change the clear color in ApplicationClass::Render to yellow.

#### File : applicationclass.cpp

```c++
bool ApplicationClass::Render() {
	// Clear the buffers to begin the scene.
	m_Direct3D->BeginScene(1.0f, 1.0f, 0, 1.0f); //Change those value


	// Present the rendered scene to the screen.
	m_Direct3D->EndScene();

	return true;
}
```


## 4. Write the video card name out to a text file.

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

	/*Code to output videocard information to a .txt file*/
	FILE* videocardInformationFile;
	char videocardName[128] = "";
	int videocardMemory = 0;
	m_Direct3D->GetVideoCardInfo(videocardName, videocardMemory);

	fopen_s(&videocardInformationFile, "videocardInformation.txt", "w");
	if (videocardInformationFile) {
		fprintf_s(videocardInformationFile, "Videocard: %s\nVideocard Memory: %i\n", videocardName, videocardMemory);
	}
	else {
		MessageBox(hwnd, L"Could not write the videocard information to the file", L"Error", MB_OK);
	}

	return true;
}
```
