# InputManager 

```C#

using UnityEngine;
using UnityEngine.InputSystem;



[DefaultExecutionOrder(-1)]
public class InputManager : Singleton<InputManager> //public class InputManager : MonoBehaviour
{
    // creating event 
    public delegate void StartTouchEvent(Vector2 position, float time);
    public event StartTouchEvent OnStartTouchEvent;
    public delegate void EndTouchEvent(Vector2 position, float time);
    public event EndTouchEvent OnEndTouchEvent;

    private TouchControls touchControls;
    
    private void Awake()
    {
        touchControls = new TouchControls();
    }
    private void OnEnable()
    {
        touchControls.Enable();
    }
    private void OnDisable()
    {
        touchControls.Disable();
    }
    private void Start()
    {
        touchControls.Touch.TouchPress.started += ctx => StartTouch(ctx);
        touchControls.Touch.TouchPress.canceled += ctx => EndTouch(ctx);
    }
    private void StartTouch(InputAction.CallbackContext context)
    {
        Debug.Log($"Touch World : {Camera.main.ScreenToWorldPoint(touchControls.Touch.TouchPosition.ReadValue<Vector2>())}");
        Debug.Log($"Touch Screen : {touchControls.Touch.TouchPosition.ReadValue<Vector2>()}");

        OnStartTouchEvent?.Invoke(touchControls.Touch.TouchPosition.ReadValue<Vector2>(), (float)context.startTime);
    }
    private void EndTouch(InputAction.CallbackContext context)
    {
        Debug.Log($"Touch Ended");
        OnEndTouchEvent?.Invoke(touchControls.Touch.TouchPosition.ReadValue<Vector2>(), (float)context.time);
    }
}
    

```
# TouchSubscriber
```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.InputSystem;

public class TouchSubscriber : MonoBehaviour
{

    private InputManager inputManager;
    private Camera cameraMain;
         
    private void Awake()
    {
        inputManager = InputManager.Instance;
        cameraMain = Camera.main;
    }
    private void OnEnable()
    {
        inputManager.OnStartTouchEvent += Move;
    }
    private void OnDisable()
    {
        inputManager.OnStartTouchEvent -= Move;
    }

    public void Move(Vector2 screenPosition, float time)
    {
        Vector3 screenCoordinate = new Vector3(screenPosition.x, screenPosition.y, cameraMain.nearClipPlane);
        Vector3 worldCordinate = cameraMain.ScreenToWorldPoint (screenCoordinate);
        worldCordinate.z = 0;
        transform.position = worldCordinate;

        //Vector3 transformIs = transform.position;
        //transformIs.x = worldCordinate.x;
        //transformIs.y = worldCordinate.y;
        //transformIs.z = gameObject.transform.position.z;
       // transform.position = transformIs;
    }
}

```
