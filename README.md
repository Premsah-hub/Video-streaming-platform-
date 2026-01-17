# Video-streaming-platform-
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.io.File;

import javafx.application.Platform;
import javafx.embed.swing.JFXPanel;
import javafx.scene.Group;
import javafx.scene.Scene;
import javafx.scene.media.Media;
import javafx.scene.media.MediaPlayer;
import javafx.scene.media.MediaView;
import javafx.util.Duration;

public class VideoPlayer extends JFrame {

    private JFXPanel fxPanel;
    private MediaPlayer mediaPlayer;

    private JButton playPauseBtn, nextBtn, prevBtn, fullscreenBtn;
    private JSlider progressSlider, volumeSlider;
    private JList<String> videoList;

    private File[] playlist;
    private int currentIndex = 0;
    private boolean isPlaying = true;
    private boolean isFullscreen = false;

    private GraphicsDevice device;

    public VideoPlayer() {
        setTitle("Mini YouTube Player");
        setSize(1100, 600);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLayout(new BorderLayout());
        getContentPane().setBackground(Color.BLACK);

        device = GraphicsEnvironment
                .getLocalGraphicsEnvironment()
                .getDefaultScreenDevice();

        fxPanel = new JFXPanel();
        add(fxPanel, BorderLayout.CENTER);

        createPlaylistPanel();
        createControlPanel();
        loadPlaylist();

        addKeyListener(new KeyAdapter() {
            public void keyPressed(KeyEvent e) {
                if (e.getKeyCode() == KeyEvent.VK_ESCAPE && isFullscreen) {
                    toggleFullscreen();
                }
            }
        });

        setFocusable(true);
        setVisible(true);
    }

    // ---------- PLAYLIST PANEL (RIGHT SIDE) ----------
    private void createPlaylistPanel() {
        DefaultListModel<String> model = new DefaultListModel<>();

        playlist = new File[] {
                new File("videos/video1.mp4"),
                new File("videos/video2.mp4"),
                new File("videos/video3.mp4")
        };

        for (File f : playlist) {
            model.addElement(f.getName());
        }

        videoList = new JList<>(model);
        videoList.setBackground(Color.DARK_GRAY);
        videoList.setForeground(Color.WHITE);
        videoList.setSelectionMode(ListSelectionModel.SINGLE_SELECTION);

        videoList.addListSelectionListener(e -> {
            if (!e.getValueIsAdjusting()) {
                currentIndex = videoList.getSelectedIndex();
                playVideo(currentIndex);
            }
        });

        JScrollPane scrollPane = new JScrollPane(videoList);
        scrollPane.setPreferredSize(new Dimension(250, 0));
        add(scrollPane, BorderLayout.EAST);
    }

    // ---------- CONTROL PANEL ----------
    private void createControlPanel() {
        JPanel controls = new JPanel();
        controls.setBackground(Color.DARK_GRAY);

        prevBtn = new JButton("⏮");
        playPauseBtn = new JButton("⏸");
        nextBtn = new JButton("⏭");
        fullscreenBtn = new JButton("⛶");

        progressSlider = new JSlider();
        progressSlider.setPreferredSize(new Dimension(300, 20));

        volumeSlider = new JSlider(0, 100, 70);
        volumeSlider.setPreferredSize(new Dimension(100, 20));

        prevBtn.addActionListener(e -> playPrevious());
        nextBtn.addActionListener(e -> playNext());
        playPauseBtn.addActionListener(e -> togglePlayPause());
        fullscreenBtn.addActionListener(e -> toggleFullscreen());

        volumeSlider.addChangeListener(e -> {
            if (mediaPlayer != null) {
                mediaPlayer.setVolume(volumeSlider.getValue() / 100.0);
            }
        });

        controls.add(prevBtn);
        controls.add(playPauseBtn);
        controls.add(nextBtn);
        controls.add(fullscreenBtn);
        controls.add(progressSlider);
        controls.add(new JLabel("Vol"));
        controls.add(volumeSlider);

        add(controls, BorderLayout.SOUTH);
    }

    // ---------- VIDEO LOGIC ----------
    private void loadPlaylist() {
        videoList.setSelectedIndex(0);
        playVideo(0);
    }

    private void playVideo(int index) {
        Platform.runLater(() -> {
            if (mediaPlayer != null) {
                mediaPlayer.stop();
            }

            Media media = new Media(playlist[index].toURI().toString());
            mediaPlayer = new MediaPlayer(media);

            MediaView mediaView = new MediaView(mediaPlayer);
            mediaView.setFitWidth(850);
            mediaView.setFitHeight(480);

            Group root = new Group(mediaView);
            Scene scene = new Scene(root);

            fxPanel.setScene(scene);

            mediaPlayer.setOnReady(() -> {
                progressSlider.setMaximum((int) media.getDuration().toSeconds());
            });

            mediaPlayer.currentTimeProperty().addListener((obs, oldTime, newTime) -> {
                if (!progressSlider.getValueIsAdjusting()) {
                    progressSlider.setValue((int) newTime.toSeconds());
                }
            });

            progressSlider.addChangeListener(e -> {
                if (progressSlider.getValueIsAdjusting()) {
                    mediaPlayer.seek(Duration.seconds(progressSlider.getValue()));
                }
            });

            mediaPlayer.play();
            isPlaying = true;
            playPauseBtn.setText("⏸");
        });
    }

    private void togglePlayPause() {
        if (mediaPlayer == null) return;

        if (isPlaying) {
            mediaPlayer.pause();
            playPauseBtn.setText("▶");
        } else {
            mediaPlayer.play();
            playPauseBtn.setText("⏸");
        }
        isPlaying = !isPlaying;
    }

    private void playNext() {
        currentIndex = (currentIndex + 1) % playlist.length;
        videoList.setSelectedIndex(currentIndex);
    }

    private void playPrevious() {
        currentIndex = (currentIndex - 1 + playlist.length) % playlist.length;
        videoList.setSelectedIndex(currentIndex);
    }

    // ---------- FULLSCREEN ----------
    private void toggleFullscreen() {
        dispose();
        isFullscreen = !isFullscreen;
        setUndecorated(isFullscreen);
        device.setFullScreenWindow(isFullscreen ? this : null);
        setVisible(true);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(VideoPlayer::new);
    }
}
