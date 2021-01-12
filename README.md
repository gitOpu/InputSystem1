
# Previus Input System
```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;


public class TouchManager : MonoBehaviour
{
	// If the touch is longer than MAX_SWIPE_TIME, we dont consider it a swipe
	public const float MAX_SWIPE_TIME = 0.5f;

	// Factor of the screen width that we consider a swipe
	// 0.17 works well for portrait mode 16:9 phone
	public const float MIN_SWIPE_DISTANCE = 0.17f;

	public static bool swipedRight = false;
	public static bool swipedLeft = false;
	public static bool swipedUp = false;
	public static bool swipedDown = false;


	// private bool debugWithArrowKeys = true;

	Vector2 startPos;
	float startTime;


	// for Double Tap ............................
	int TapCount = 0;
	public float MaxDubbleTapTime = 0.2f;
	float NewTime;
	public static bool isDoubleTapped = false;


	void LateUpdate()
	{
		DoubleTap();

		DetectSwipe();

		if (Input.GetKey(KeyCode.A))
		{
			Debug.Log("Working-----");
		}
	}

	public void DetectSwipe()
	{
		swipedRight = false;
		swipedLeft = false;
		swipedUp = false;
		swipedDown = false;

		if (Input.touches.Length > 0)
		{
			Touch t;
			t = Input.GetTouch(0);
			if (Input.touches.Length == 2)
			{
				t = Input.GetTouch(1);
			}
			if (t.phase == TouchPhase.Began)
			{
				startPos = new Vector2(t.position.x / (float)Screen.width, t.position.y / (float)Screen.width);
				startTime = Time.time;
			}
			if (t.phase == TouchPhase.Ended)
			{
				if (Time.time - startTime > MAX_SWIPE_TIME) // press too long
					return;

				Vector2 endPos = new Vector2(t.position.x / (float)Screen.width, t.position.y / (float)Screen.width);

				Vector2 swipe = new Vector2(endPos.x - startPos.x, endPos.y - startPos.y);

				if (swipe.magnitude < MIN_SWIPE_DISTANCE) // Too short swipe
					return;

				if (Mathf.Abs(swipe.x) > Mathf.Abs(swipe.y))
				{ // Horizontal swipe
					if (swipe.x > 0)
					{
						swipedRight = true;
						Debug.Log("SwipedRight");
					}
					else
					{
						swipedLeft = true;
						Debug.Log("swipedLeft");
					}
				}
				else
				{ // Vertical swipe
					if (swipe.y > 0)
					{
						swipedUp = true;
						//Debug.Log("swipedUp");
					}
					else
					{
						swipedDown = true;
						Debug.Log("swipedDown");
					}
				}
			}
		}


		//if (debugWithArrowKeys)
		//{
		//	swipedDown = swipedDown || Input.GetKeyDown(KeyCode.DownArrow);
		//	swipedUp = swipedUp || Input.GetKeyDown(KeyCode.UpArrow);
		//	swipedRight = swipedRight || Input.GetKeyDown(KeyCode.RightArrow);
		//	swipedLeft = swipedLeft || Input.GetKeyDown(KeyCode.LeftArrow);
		//}
	}

	public void DoubleTap()
	{
		isDoubleTapped = false;

		if (Input.touchCount == 2)
		{
			Touch touch = Input.GetTouch(1);

			if (touch.phase == TouchPhase.Ended)
			{
				TapCount += 1;
			}

			if (TapCount == 1)
			{

				NewTime = Time.time + MaxDubbleTapTime;
			}
			else if (TapCount == 2 && Time.time <= NewTime)
			{
				TapCount = 0;
				isDoubleTapped = true;
				//Debug.Log("Double Tapped True");
			}

		}
		if (Time.time > NewTime)
		{
			TapCount = 0;
		}

	}


}

```
# New Inpt System
### InputManager 

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
### TouchSubscriber
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
