#player
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
