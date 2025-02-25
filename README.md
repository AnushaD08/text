@Service
public class FollowService {
    @Autowired
    private FollowRepository followRepository;
    @Autowired
    private UserRepository userRepository;

    public void followUser(Long followerId, Long followingId) {
        User follower = userRepository.findById(followerId).orElseThrow(() -> new RuntimeException("Follower not found"));
        User following = userRepository.findById(followingId).orElseThrow(() -> new RuntimeException("Following not found"));

        if (!followRepository.existsByFollowerAndFollowing(follower, following)) {
            Follow follow = new Follow();
            follow.setFollower(follower);
            follow.setFollowing(following);
            followRepository.save(follow);
        }
    }

    public void unfollowUser(Long followerId, Long followingId) {
        User follower = userRepository.findById(followerId).orElseThrow(() -> new RuntimeException("Follower not found"));
        User following = userRepository.findById(followingId).orElseThrow(() -> new RuntimeException("Following not found"));

        followRepository.deleteByFollowerAndFollowing(follower, following);
    }
}

@RestController
@RequestMapping("/follow")
public class FollowController {
    @Autowired
    private FollowService followService;

    @PostMapping("/{followerId}/{followingId}")
    public ResponseEntity<String> follow(@PathVariable Long followerId, @PathVariable Long followingId) {
        followService.followUser(followerId, followingId);
        return ResponseEntity.ok("Followed successfully");
    }

    @DeleteMapping("/{followerId}/{followingId}")
    public ResponseEntity<String> unfollow(@PathVariable Long followerId, @PathVariable Long followingId) {
        followService.unfollowUser(followerId, followingId);
        return ResponseEntity.ok("Unfollowed successfully");
    }
}
