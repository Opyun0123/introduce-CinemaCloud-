private static final List<String> ALL_TAGS = Arrays.asList(
    "드라마", "로맨스", "코미디", "스릴러", "미스터리", 
    "호러", "액션", "SF", "판타지", "다큐멘터리", "아드벤처", 
    "우화", "다문화", "가족", "음악", "해적", "심리적", "비극적", 
    "극발", "서스펜스", "정서적", "사랑", "운명", "실화", "체험적", 
    "형이상학적", "패러디", "반전", "서정적", "상상력", "유머", 
    "혼란", "노스텔지어", "실험적", "미니멀리즘", "예술적", "하이테크", 
    "가상 현실", "미래적", "고전", "전쟁", "역사적", "대체 역사", 
    "대로", "북전", "복신", "반운용", "가우에포", "구초", "문화", 
    "시험적", "시간", "강화", "강력", "통제", "결여", "방언", "테마"
);

public List<Movie> getRecommendations(Long profileId) {
    Profile profile = profileRepository.findById(profileId)
            .orElseThrow(() -> new IllegalArgumentException("Invalid profile ID"));

    Map<String, Integer> profileTags = parseJsonToMap(profile.getProfileVector());

    List<Movie> recommendations = new ArrayList<>();
    for (Movie movie : movieRepository.findAll()) {
        List<String> movieTags = parseJsonToList(movie.getTags());
        double similarity = calculateCosineSimilarity(profileTags, movieTags);
        if (similarity > 0.1) {  // 임계치 예시
            recommendations.add(movie);
        }
    }
    return recommendations;
}

private double calculateCosineSimilarity(Map<String, Integer> profileTags, List<String> movieTags) {
    int[] profileVector = createVectorFromTags(profileTags);
    int[] movieVector = createVectorFromTags(movieTags);

    double dotProduct = 0.0;
    double profileMagnitude = 0.0;
    double movieMagnitude = 0.0;

    for (int i = 0; i < profileVector.length; i++) {
        dotProduct += profileVector[i] * movieVector[i];
        profileMagnitude += Math.pow(profileVector[i], 2);
        movieMagnitude += Math.pow(movieVector[i], 2);
    }

    profileMagnitude = Math.sqrt(profileMagnitude);
    movieMagnitude = Math.sqrt(movieMagnitude);

    if (profileMagnitude == 0 || movieMagnitude == 0) {
        return 0.0;  
    }

    return dotProduct / (profileMagnitude * movieMagnitude);
}
