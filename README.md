#duan1
using UnityEngine;
using UnityEngine.Rendering;

public class Player : MonoBehaviour
{
    [SerializeField] 
    private float JumpForce = 10f;
    private Rigidbody2D rb;
    private bool isGrounded;
    [SerializeField]
    private Transform groundCheck;
    [SerializeField]
    private float groundCheckRadius = 0.2f;
    [SerializeField]
    private LayerMask groundLayer;
    void Start()
    {
        rb = GetComponent<Rigidbody2D>();
    }

    void Update()
    {
        isGrounded = CheckIfGrounded();
        HandleJump();
    }
    private bool CheckIfGrounded()
    {
        return Physics2D.OverlapCircle(
            groundCheck.position,     // Tọa độ tâm vòng tròn kiểm tra
            groundCheckRadius,        // Bán kính vòng tròn
            groundLayer               // Chỉ va chạm với layer mặt đất
        );
    }
    private void OnDrawGizmosSelected()
    {
        Gizmos.color = Color.red;
        Gizmos.DrawWireSphere(groundCheck.position, groundCheckRadius);
    }
    private void HandleJump()
    {
        if (Input.GetKeyDown(KeyCode.Space) && isGrounded)
        {
            rb.linearVelocity = new Vector2(rb.linearVelocity.x, JumpForce);
        }
    }
}

public class Obstacle : MonoBehaviour
{
    public float leftBoundary = -10f; // Left boundary for the obstacle
    // Start is called once before the first execution of Update after the MonoBehaviour is created
    void Start()
    {

    }

    // Update is called once per frame
    void Update()
    {
        MoveObstacle();
        FaceLeft();
    }
    private void MoveObstacle()
    {
        transform.position += Vector3.left * GameManager.instance.GetGameSpeed() * Time.deltaTime;
        if(transform.position.x < leftBoundary)
        {
            Destroy(gameObject); // Destroy the obstacle if it goes beyond the left boundary
        }
    }
    private void FaceLeft()
    {
        Vector3 scale = transform.localScale;
        scale.x = Mathf.Abs(scale.x) * -1;
        transform.localScale = scale;
    }
    private void OnTriggerEnter2D(Collider2D collision)
    {
        if (collision.CompareTag("Player"))
        {
            GameManager.instance.GameOver(); // Call the GameOver method from GameManager
            gameObject.SetActive(false); // Deactivate the obstacle instead of destroying it
        }
    }
}

public class Parallax : MonoBehaviour
{
    private Material material;
    [SerializeField] 
    private float parallaxFactor = 0.01f;
    private float offset;
    // Start is called once before the first execution of Update after the MonoBehaviour is created
    void Start()
    {
        material = GetComponent<Renderer>().material;
    }

    // Update is called once per frame
    void Update()
    {
        ParallaxScroll();
    }
    private void ParallaxScroll()
    {
        float speed = GameManager.instance.GetGameSpeed() * parallaxFactor;
        offset += Time.deltaTime * speed;
        material.SetTextureOffset("_MainTex", Vector2.right * offset);
    }    
}

public class MainMenu : MonoBehaviour
{
   public void LoadGame()
    {
        SceneManager.LoadScene("Game"); // Load the game scene
    }
   public void ExitGame()
   {
        Application.Quit();
   }
}

public class Spawner : MonoBehaviour
{
    [SerializeField] private GameObject[] obstacles;
    [SerializeField] private Transform lowpos;
    private float timer = 0f;
    [SerializeField] private float spawnRate = 2f;
    // Update is called once per frame
    void Update()
    {
        timer += Time.deltaTime;
        if (timer >= spawnRate)
        {
            SpawnObstacle();
            timer = 0;
        }
    }
    private void SpawnObstacle()
    {
        int index = Random.Range(0, obstacles.Length);
        if (index == 0 || index ==1)
        {
            GameObject obstacle = Instantiate(obstacles[index], lowpos.position, Quaternion.identity);
        }
    }
}

public class GameManager : MonoBehaviour
{
    public static GameManager instance;
    private float gamespeed = 5f;
    [SerializeField] private float speedIncrease = 0.15f;
    [SerializeField] private TextMeshProUGUI scoreText;
    private float score = 0f;
    [SerializeField] private GameObject scoreTextObject;
    [SerializeField] private GameObject gameStart;
    [SerializeField] private GameObject gameOver;
    private bool isGameOver = false;
    private void Awake()
    {
        if (instance == null)
        {
            instance = this;
        }
}
    // Start is called once before the first execution of Update after the MonoBehaviour is created
    void Start()
    {
        StartGame();
    }

    // Update is called once per frame
    void Update()
    {
        HandleStartGameInput();
        HandleExitToMenuInput();
        if (!isGameOver)
        {
            UpdateGameSpeed();
            UpdateScore();
        }    
    }
    public float GetGameSpeed()
    {
        return gamespeed;
    }
    private void UpdateGameSpeed()
    {
        gamespeed += speedIncrease * Time.deltaTime;
    }
    private void UpdateScore()
    {
        score += Time.deltaTime * 10;
        scoreText.text = "Score: " + Mathf.FloorToInt(score);
    }
    private void StartGame()
    {
        Time.timeScale = 0f;
        scoreTextObject.SetActive(false);
        gameStart.SetActive(true);
        gameOver.SetActive(false);
    }   
    private void HandleStartGameInput()
    {
        if(Input.GetKeyDown(KeyCode.Return))
        {
            Time.timeScale = 1f;
            scoreTextObject.SetActive(true);
            gameStart.SetActive(false);
        }
    }  
    public void GameOver()
    {
        isGameOver = true;
        gameOver.SetActive(true);
        Time.timeScale = 0f;
        StartCoroutine(ReLoadScene());
    }
    private IEnumerator ReLoadScene()
    {
        yield return new WaitForSecondsRealtime(1f);
        SceneManager.LoadScene(SceneManager.GetActiveScene().name);
    }
    private void HandleExitToMenuInput()
    {
        if (Input.GetKeyDown(KeyCode.Escape))
        {
            LoadMenu();
        }
    }
    public void LoadMenu()
    {
        Time.timeScale = 1f; // reset time
        SceneManager.LoadScene("Menu"); // đổi "Menu" thành tên scene menu của bạn
    }
}

public class PlayerCollsion : MonoBehaviour
{
    private GameManager gameManager;
    private void Awake()
    {
        gameManager = FindAnyObjectByType<GameManager>();
    }
    private void OnTriggerEnter2D(Collider2D collision)
    {
        if (collision.CompareTag("Coin"))
        {
           Debug.Log("Hit Coin");
            Destroy(collision.gameObject);
        }
    }
}
