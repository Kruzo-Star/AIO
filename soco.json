<?php

function fetchServerURL($roomNum) {
    $url = "https://json.vnres.co/room/{$roomNum}/detail.json";
    $response = file_get_contents($url);

    if ($response !== false) {
        if (preg_match('/detail\((.*)\)/', $response, $matches)) {
            $jsonData = json_decode($matches[1], true);
            if (isset($jsonData['code']) && $jsonData['code'] === 200) {
                $stream = $jsonData['data']['stream'];
                return [
                    'm3u8' => $stream['m3u8'] ?? null,
                    'hdM3u8' => $stream['hdM3u8'] ?? null,
                ];
            }
        }
    }

    return ['m3u8' => null, 'hdM3u8' => null];
}

function fetchMatches($timeData, $mainReferer, $userAgent) {
    $url = "https://json.vnres.co/match/matches_{$timeData}.json";

    $options = [
        "http" => [
            "header" => "referer: {$mainReferer}\r\n" .
                        "user-agent: {$userAgent}\r\n" .
                        "origin: https://json.vnres.co\r\n"
        ]
    ];

    $context = stream_context_create($options);
    $response = file_get_contents($url, false, $context);

    $dailyMatches = [];

    if ($response !== false) {
        if (preg_match('/matches_\d+\((.*)\)/', $response, $matches)) {
            $jsonData = json_decode($matches[1], true);
            if (isset($jsonData['code']) && $jsonData['code'] === 200) {
                $matches = $jsonData['data'];
                $currentTimeSeconds = time();
                $tenMinutesLater = $currentTimeSeconds + 600;

                foreach ($matches as $match) {
                    try {
                        $leagueName = $match['subCateName'];
                        $homeTeamName = $match['hostName'];
                        $homeTeamLogo = $match['hostIcon'];
                        $awayTeamName = $match['guestName'];
                        $awayTeamLogo = $match['guestIcon'];

                        $matchTime = intval($match['matchTime'] / 1000);
                        $matchStatus = ($currentTimeSeconds >= $matchTime || $tenMinutesLater > $matchTime)
                            ? "live"
                            : "vs";

                        $serversList = [];
                        if ($matchStatus === "live") {
                            foreach ($match['anchors'] as $anchor) {
                                $serverRoom = $anchor['anchor']['roomNum'];
                                $streamData = fetchServerURL($serverRoom);

                                if ($streamData['m3u8']) {
                                    $serversList[] = [
                                        'name' => "Soco SD",
                                        'stream_url' => $streamData['m3u8'],
                                        'referer' => $mainReferer,
                                    ];
                                }
                                if ($streamData['hdM3u8']) {
                                    $serversList[] = [
                                        'name' => "Soco HD",
                                        'stream_url' => $streamData['hdM3u8'],
                                        'referer' => $mainReferer,
                                    ];
                                }
                            }
                        }

                        $dailyMatches[] = [
                            'match_time' => (string)$matchTime,
                            'match_status' => $matchStatus,
                            'home_team_name' => $homeTeamName,
                            'home_team_logo' => $homeTeamLogo,
                            'away_team_name' => $awayTeamName,
                            'away_team_logo' => $awayTeamLogo,
                            'league_name' => $leagueName,
                            'servers' => $serversList,
                        ];
                    } catch (Exception $e) {
                        error_log($e->getMessage());
                    }
                }
            }
        }
    }

    return $dailyMatches;
}

    $mainReferer = "https://socolivev.co/";
    $userAgent = $_SERVER['HTTP_USER_AGENT'] ?? "Default-User-Agent";

    $currentDate = date('Ymd');
    $nextDay = date('Ymd', strtotime('+1 day'));
    $yesterday = date('Ymd', strtotime('-1 day'));

    $matchTimes = [$yesterday, $currentDate, $nextDay];

    $allMatches = [];
    foreach ($matchTimes as $time) {
        $allMatches = array_merge($allMatches, fetchMatches($time, $mainReferer, $userAgent));
    }

    header('Content-Type: application/json');
    echo json_encode($allMatches, JSON_PRETTY_PRINT);
    exit;

?>
