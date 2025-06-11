## React Native Code

```js
import React, { useEffect } from 'react';
import { View, Text, StyleSheet } from 'react-native';
import Svg, { Path, Text as SvgText, G } from 'react-native-svg';
import Animated, {
  useSharedValue,
  useAnimatedProps,
  withTiming,
} from 'react-native-reanimated';

const AnimatedPath = Animated.createAnimatedComponent(Path);

const polarToCartesian = (cx, cy, r, angleInDegrees) => {
  const angleInRadians = (angleInDegrees - 90) * Math.PI / 180.0;
  return {
    x: cx + r * Math.cos(angleInRadians),
    y: cy + r * Math.sin(angleInRadians),
  };
};

const createArcPath = (cx, cy, r, startAngle, sweepAngle) => {
  const endAngle = startAngle + sweepAngle;
  const start = polarToCartesian(cx, cy, r, startAngle);
  const end = polarToCartesian(cx, cy, r, endAngle);
  const largeArcFlag = sweepAngle > 180 ? 1 : 0;

  return [
    'M', start.x, start.y,
    'A', r, r, 0, largeArcFlag, 1, end.x, end.y,
  ].join(' ');
};

const DonutChart = ({ data, radius = 80, strokeWidth = 25, duration = 1000 }) => {
  const total = data.reduce((sum, item) => sum + item.amount, 0);
  const center = radius + strokeWidth;
  const progress = useSharedValue(0);

  useEffect(() => {
    progress.value = withTiming(1, { duration });
  }, []);

  return (
    <View style={styles.chartContainer}>
      <Svg width={center * 2} height={center * 2}>
        <G rotation="-90" originX={center} originY={center}>
          {data.map((item, index) => {
            const value = item.amount / total;

            const animatedProps = useAnimatedProps(() => {
              const sweep = value * 360 * progress.value;

              // Calculate startAngle from previous slices
              let startAngle = 0;
              for (let i = 0; i < index; i++) {
                startAngle += (data[i].amount / total) * 360;
              }

              const path = createArcPath(center, center, radius, startAngle, sweep);
              return { d: path };
            });

            return (
              <AnimatedPath
                key={index}
                animatedProps={animatedProps}
                stroke={item.color}
                strokeWidth={strokeWidth}
                fill="none"
                strokeLinecap="round"
              />
            );
          })}

          {/* Labels */}
          {data.map((item, index) => {
            const percent = Math.round((item.amount / total) * 100);
            if (percent < 3) return null; // skip small slices

            // Calculate midAngle from static data
            let startAngle = 0;
            for (let i = 0; i < index; i++) {
              startAngle += (data[i].amount / total) * 360;
            }
            const sweepAngle = (item.amount / total) * 360;
            const midAngle = startAngle + sweepAngle / 2;
            const labelPos = polarToCartesian(center, center, radius + strokeWidth / 2 + 10, midAngle);

            return (
              <SvgText
                key={`label-${index}`}
                x={labelPos.x}
                y={labelPos.y}
                fill="#000"
                fontSize="12"
                fontWeight="bold"
                textAnchor="middle"
              >
                {percent}%
              </SvgText>
            );
          })}
        </G>
      </Svg>
      <View style={styles.centerText}>
        <Text style={styles.total}>₹ {total}</Text>
        <Text style={styles.caption}>Total</Text>
      </View>
    </View>
  );
};

export default function App() {
  const data = [
    { category: 'Food', amount: 3000, color: '#FF6B6B' },
    { category: 'Rent', amount: 8000, color: '#4ECDC4' },
    { category: 'Transport', amount: 1500, color: '#FFD93D' },
    { category: 'Other', amount: 2500, color: '#1A535C' },
  ];

  return (
    <View style={styles.appContainer}>
      <DonutChart data={data} />
    </View>
  );
}

const styles = StyleSheet.create({
  appContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  chartContainer: {
    justifyContent: 'center',
    alignItems: 'center',
    position: 'relative',
  },
  centerText: {
    position: 'absolute',
    alignItems: 'center',
  },
  total: {
    fontSize: 22,
    fontWeight: 'bold',
  },
  caption: {
    fontSize: 14,
    color: '#888',
  },
});

```
