@Entity
@Table(name = "reports")
public class Report {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    @JoinColumn(name = "post_id", nullable = false)
    private Post post;

    @ManyToOne
    @JoinColumn(name = "user_id", nullable = false)
    private User reportedBy;

    @Column(nullable = false)
    private String reason;

    @Enumerated(EnumType.STRING)
    private ReportStatus status = ReportStatus.PENDING; // PENDING, REVIEWED, ACTION_TAKEN

    @CreationTimestamp
    private LocalDateTime reportedAt;
}
🔹 Report Repository
java
Copy
Edit
@Repository
public interface ReportRepository extends JpaRepository<Report, Long> {
    List<Report> findByStatus(ReportStatus status);
}
🔹 Report Service
java
Copy
Edit
@Service
public class ReportService {
    @Autowired private ReportRepository reportRepository;
    @Autowired private PostRepository postRepository;
    @Autowired private UserRepository userRepository;

    public void reportPost(Long postId, Long userId, String reason) {
        Post post = postRepository.findById(postId)
                .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND, "Post not found"));
        User user = userRepository.findById(userId)
                .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND, "User not found"));

        Report report = new Report();
        report.setPost(post);
        report.setReportedBy(user);
        report.setReason(reason);
        reportRepository.save(report);
    }
}
🔹 Report Controller
java
Copy
Edit
@RestController
@RequestMapping("/reports")
public class ReportController {
    @Autowired private ReportService reportService;

    @PostMapping("/{postId}")
    public ResponseEntity<String> reportPost(@PathVariable Long postId, @RequestParam Long userId, @RequestParam String reason) {
        reportService.reportPost(postId, userId, reason);
        return ResponseEntity.status(HttpStatus.CREATED).body("Post reported successfully");
    }
}
2️⃣ View Reported Posts (Admin Only) API (/admin/reports)
🔹 ReportService
java
Copy
Edit
@Service
public class AdminReportService {
    @Autowired private ReportRepository reportRepository;

    public List<Report> getAllReportedPosts() {
        return reportRepository.findByStatus(ReportStatus.PENDING);
    }
}
🔹 Report Controller
java
Copy
Edit
@RestController
@RequestMapping("/admin/reports")
@PreAuthorize("hasRole('ADMIN')")
public class AdminReportController {
    @Autowired private AdminReportService adminReportService;

    @GetMapping
    public ResponseEntity<List<Report>> getAllReportedPosts() {
        return ResponseEntity.ok(adminReportService.getAllReportedPosts());
    }
}
3️⃣ Act on a Report (Admin Only) API (/admin/reports/{reportId}/action)
🔹 Report Service
java
Copy
Edit
@Service
public class AdminActionService {
    @Autowired private ReportRepository reportRepository;

    public void actOnReport(Long reportId, ReportStatus newStatus) {
        Report report = reportRepository.findById(reportId)
                .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND, "Report not found"));
        report.setStatus(newStatus);
        reportRepository.save(report);
    }
}
🔹 Report Controller
java
Copy
Edit
@RestController
@RequestMapping("/admin/reports")
@PreAuthorize("hasRole('ADMIN')")
public class AdminActionController {
    @Autowired private AdminActionService adminActionService;

    @PostMapping("/{reportId}/action")
    public ResponseEntity<String> actOnReport(@PathVariable Long reportId, @RequestParam ReportStatus status) {
        adminActionService.actOnReport(reportId, status);
        return ResponseEntity.ok("Report status updated");
    }
}
