Like Repository
java
Copy
Edit
@Repository
public interface LikeRepository extends JpaRepository<Like, Long> {
    void deleteByUserAndPost(User user, Post post);
    boolean existsByUserAndPost(User user, Post post);
}
Like Service
java
Copy
Edit
@Service
public class LikeService {
    @Autowired
    private LikeRepository likeRepository;
    @Autowired
    private UserRepository userRepository;
    @Autowired
    private PostRepository postRepository;

    public void likePost(Long userId, Long postId) {
        User user = userRepository.findById(userId).orElseThrow(() -> new RuntimeException("User not found"));
        Post post = postRepository.findById(postId).orElseThrow(() -> new RuntimeException("Post not found"));

        if (!likeRepository.existsByUserAndPost(user, post)) {
            Like like = new Like();
            like.setUser(user);
            like.setPost(post);
            likeRepository.save(like);
        }
    }

    public void unlikePost(Long userId, Long postId) {
        User user = userRepository.findById(userId).orElseThrow(() -> new RuntimeException("User not found"));
        Post post = postRepository.findById(postId).orElseThrow(() -> new RuntimeException("Post not found"));

        likeRepository.deleteByUserAndPost(user, post);
    }
}
Like Controller
java
Copy
Edit
@RestController
@RequestMapping("/likes")
public class LikeController {
    @Autowired
    private LikeService likeService;

    @PostMapping("/{userId}/{postId}")
    public ResponseEntity<String> likePost(@PathVariable Long userId, @PathVariable Long postId) {
        likeService.likePost(userId, postId);
        return ResponseEntity.ok("Post liked");
    }

    @DeleteMapping("/{userId}/{postId}")
    public ResponseEntity<String> unlikePost(@PathVariable Long userId, @PathVariable Long postId) {
        likeService.unlikePost(userId, postId);
        return ResponseEntity.ok("Post unliked");
    }
}
